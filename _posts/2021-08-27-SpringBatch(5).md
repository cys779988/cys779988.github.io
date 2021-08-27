---
title:  "spring batch(5) Scope & Job Parameter"
excerpt: spring batch(5)
categories:
  - spring
---

## Chunk
- 스프링배치에서의 Chunk란 데이터 덩어리로 작업할 때 각 커밋 사이에 처리되는 row 수를 얘기한다.
- 즉 Chunk 지향 처리란 한번에 하나씩 데이터를 읽어 Chunk 라는 덩어리를 만든 뒤, Chunk 단위로 트랜잭션을 다루는 것을 의미
- Chunk 단위로 트랜잭션을 수행하기 때문에 실패한 경우엔 해당 Chunk 만큼만 롤백, 이전에 커밋된 트랜잭션 범위까지는 반영이 됨

<img src="https://cys779988.github.io/assets/img/springbatch-5.PNG">

- Reader에서 데이터를 읽어옴
- 읽어온 데이터를 Processor에서 가공
- 가공된 데이터들을 별도의 공간에 모은 뒤, Chunk 단위만큼 쌓이게 되면 Writer에 전달하고 Writer는 일괄 저장함

#### Reader와 Processor에서는 1건씩 다뤄지고, Writer에서는 Chunk 단위로 처리됨

## ChunkOrientedTasklet

-   ```ChunkOrientedTasklet<I>(ChunkProvider<I>, ChunkProcessor<I>)```  
-   ```ChunkProvider.provide()```  로 Reader에서 Chunk size만큼 데이터를 가져옴
-   ```ChunkProvider.process()```  에서 Reader로 받은 데이터를 가공(Processor)하고 저장(Writer)


  
```java

public class SimpleChunkProvider<I> implements ChunkProvider<I> {

	protected final Log logger = LogFactory.getLog(getClass());

	protected final ItemReader<? extends I> itemReader;

	private final MulticasterBatchListener<I, ?> listener = new MulticasterBatchListener<>();

	private final RepeatOperations repeatOperations;

	public SimpleChunkProvider(ItemReader<? extends I> itemReader, RepeatOperations repeatOperations) {
		this.itemReader = itemReader;
		this.repeatOperations = repeatOperations;
	}
  
  ...
  @Override
	public Chunk<I> provide(final StepContribution contribution) throws Exception {

		final Chunk<I> inputs = new Chunk<>();
		repeatOperations.iterate(new RepeatCallback() {     //반복자

			@Override
			public RepeatStatus doInIteration(final RepeatContext context) throws Exception {
				I item = null;
				Timer.Sample sample = Timer.start(Metrics.globalRegistry);
				String status = BatchMetrics.STATUS_SUCCESS;
				try {
					item = read(contribution, inputs);      //Reader.read()
				}
				catch (SkipOverflowException e) {
					// read() tells us about an excess of skips by throwing an
					// exception
					status = BatchMetrics.STATUS_FAILURE;
					return RepeatStatus.FINISHED;
				}
				finally {
					stopTimer(sample, contribution.getStepExecution(), status);
				}
				if (item == null) {
					inputs.setEnd();
					return RepeatStatus.FINISHED;
				}
				inputs.add(item);     //반복자 끝날 때까지 inputs에 추가
				contribution.incrementReadCount();
				return RepeatStatus.CONTINUABLE;
			}

		});

		return inputs;

	}
}
```  

- input이 ChunkSize만큼 쌓일 때까지 read() 호출
- read() 는 실제로   ```ItemReader.read```  를 호출한다.
- 즉  provide()는   ```ItemReader.read```  에서 1건씩 데이터를 조회해 ChunkSize 만큼 데이터를 쌓음

## SimpleChunkProcessor
- ChunkProcessor가 Processor와 Writer 로직을 담고 있음
- SimpleChunkProcessor가 ChunkProcessor를 구현

  
```java

public class SimpleChunkProcessor<I, O> implements ChunkProcessor<I>, InitializingBean {

	private ItemProcessor<? super I, ? extends O> itemProcessor;

	private ItemWriter<? super O> itemWriter;

	private final MulticasterBatchListener<I, O> listener = new MulticasterBatchListener<>();
  
  ...
  
	@Override
	public final void process(StepContribution contribution, Chunk<I> inputs) throws Exception {

		// Allow temporary state to be stored in the user data field
		initializeUserData(inputs);

		// If there is no input we don't have to do anything more
		if (isComplete(inputs)) {
			return;
		}

		// Make the transformation, calling remove() on the inputs iterator if
		// any items are filtered. Might throw exception and cause rollback.
		Chunk<O> outputs = transform(contribution, inputs);

		// Adjust the filter count based on available data
		contribution.incrementFilterCount(getFilterCount(inputs, outputs));

		// Adjust the outputs if necessary for housekeeping purposes, and then
		// write them out...
		write(contribution, inputs, getAdjustedOutputs(inputs, outputs));

	}
}
```  

-   ```Chunk<I> inputs```  를 파라미터로 받음(이 데이터는 앞의   ```chunkProvider.provide()```  에서 받은 ChunkSize만큼 쌓인 item)
-   ```transform()```  에서는 전달 받은 inputs을 doProcess()로 전달하고 변환값을 받음
-   ```transform()```  을 통해 가공된 대량의 데이터는   ```write()```  를 통해 일괄 저장됨(write()는 저장이 될 수도 있고, 외부 API로 전송할 수도 있음. ItemWriter 구현방식에 따라 달라짐)
- transform()은 반복문을 통해 doProcess()를 호출하는데 ItemProcessor의 process()를 사용함

  
```java
	protected Chunk<O> transform(StepContribution contribution, Chunk<I> inputs) throws Exception {
		Chunk<O> outputs = new Chunk<>();
		for (Chunk<I>.ChunkIterator iterator = inputs.iterator(); iterator.hasNext();) {
			final I item = iterator.next();
			O output;
			Timer.Sample sample = BatchMetrics.createTimerSample();
			String status = BatchMetrics.STATUS_SUCCESS;
			try {
				output = doProcess(item);
			}
			catch (Exception e) {
				/*
				 * For a simple chunk processor (no fault tolerance) we are done
				 * here, so prevent any more processing of these inputs.
				 */
				inputs.clear();
				status = BatchMetrics.STATUS_FAILURE;
				throw e;
			}
			finally {
				stopTimer(sample, contribution.getStepExecution(), "item.process", status, "Item processing");
			}
			if (output != null) {
				outputs.add(output);
			}
			else {
				iterator.remove();
			}
		}
		return outputs;
	}
```  

- doProcess() 를 처리하는데 만약 ItemProcessor가 없다면 item을 그대로 반환하고 있다면 ItemProcessor의 process()로 가공하여 반환
- 가공된 데이터들은 SimpleChunkProcessor의 doWrite()를 호출하여 일괄 처리

  
```
	protected final void doWrite(List<O> items) throws Exception {

		if (itemWriter == null) {
			return;
		}

		try {
			listener.beforeWrite(items);
			writeItems(items);
			doAfterWrite(items);
		}
		catch (Exception e) {
			doOnWriteError(e, items);
			throw e;
		}

	}
```  

## Page Size VS Chunk Size
- Page Size : 한번에 조회할 Item의 양
- Chunk Size : 한번에 처리될 트랜잭션 단위
- Page Size가 10이고, Chunk Size가 50이라면 ItemReader에서 Page 조회가 5번 일어나면 1번의 트랜잭션이 발생하여 Chunk가 처리됨
- 한번의 트랜잭션 처리를 위해 5번의 쿼리 조회가 발생하기 때문에 성능상 이슈가 발생할 수 있다. 그래서 2개 값을 일치시키는 것이 보편적으로 성능향상을 위한 좋은방법

