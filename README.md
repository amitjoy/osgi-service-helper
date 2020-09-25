# OSGi Service Helper

This comprises an utility class to safely consume OSGi services using low-level APIs and that is used to safely retrieve target service instances. The release of target service objects can be performed automatically if used with try-with-resources block. Otherwise, <b>close()</b> must be invoked that releases the service objects and associated ServiceTracker. it tries to follow the best practice guidelines inherently that developers don't have to worry at all. In addition, we can also make sure that the source base would be free from resource and memory leak interferences and it could also work in the concurrent programming environment safely.

Usage 1:

```java
 try (final ServiceSupplier<MyService> serviceSupplier = ServiceSupplier.supply(MyService.class)) {
 	   final Stream<MyService> stream = serviceSupplier.get();
 }
```

Usage 2:

```java
 try (final ServiceSupplier<MyService> serviceSupplier = ServiceSupplier.supply(MyService.class, filter)) {
 	   final Stream<MyService> stream = serviceSupplier.get();
 }
```

Usage 3:

```java
 final Collection<MyService> refs = ServiceSupplier.references(MyService.class, filter)
 		.collect(toCollection());
 for (final ServiceReference<MyService> ref : refs) {
 	try (final ServiceSupplier serviceSupplier = ServiceSupplier.supply(ref)) {
 		final Stream<MyService> stream = serviceSupplier.get();
 		final Optional<MyService> service = stream.findFirst();
 	}
 }
```

 Usage 4:

```java
 final TrackerSupplier<MyService> trackerSupplier = ServiceSupplier.supplyWithTracker(MyService.class, filter);

 final BiConsumer<ServiceReference<MyService>, MyService> onAddedAction = (MyService::doA).andThen(MyService::doB);
 final BiConsumer<ServiceReference<MyService>, MyService> onRemovalAction = (MyService::removeA).andThen(MyService::removeB);
 final BiConsumer<ServiceReference<MyService>, MyService> onModifiedAction = MyService::update;

 final ServiceCallback<MyService> callback = ServiceCallbackSupplier.create()
 								    .onAdded(onAddedAction)
 								    .onModified(onModifiedAction)
 								    .onRemoved(onRemovalAction)
 								    .get();
 
 final ServiceSupplier<MyService> serviceSupplier = trackerSupplier.shouldWait(false).withCallback(callback).get();
 final Stream<MyService> stream = serviceSupplier.get();

 serviceSupplier.close(); //call it when service tracking is not required at all
```

Usage 5 (Fluent API):
 
 ```java
 final ServiceSupplier<MyService> serviceSupplier = ServiceSupplier.supplyWithTracker(MyService.class, null)
 							          .shouldWait(true)
							                  .timeout(5, SECONDS)
							                  .withCallback(
								             ServiceCallbackSupplier
								               .<MyService>create()
								               .onAdded((r, s) -> s.doA())
								               .onRemoved(MyService::removeA)
								               .get())
							                  .get();

 final Stream<MyService> stream = serviceSupplier.get();

 serviceSupplier.close(); //call it when service tracking is not required at all
```
