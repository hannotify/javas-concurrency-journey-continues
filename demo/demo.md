# Demo 1 - SingleWaiterRestaurant

*(tag `0-deep-dive-demo-start`)*

## SHOW: Restaurant interface

## CREATE: MainRestaurant

```java
void main() {
    Restaurant restaurant = new SingleWaiterRestaurant();
    restaurant.announceMenu();
}
```

## CREATE: SingleWaiterRestaurant

```java
public class SingleWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws Exception {
        Waiter elmo = new Waiter("Elmo");

        Course starter = elmo.announceCourse(CourseType.STARTER);
        Course main = elmo.announceCourse(CourseType.MAIN);
        Course dessert = elmo.announceCourse(CourseType.DESSERT);

        return new MultiCourseMeal(starter, main, dessert);
    }
}
```
## RUN: MainRestaurant

## SHOW: Waiter, Chef, Course, Ingredient

# Demo 2 - MultiWaiterThreadsRestaurant

*(tag `1-deep-dive-created-single-waiter-restaurant`)*

## CREATE: MultiWaiterThreadsRestaurant, WaiterAnnounceCourseThread

```java
public class MultiWaiterThreadsRestaurant implements Restaurant {
    class WaiterAnnounceCourseThread extends Thread {
        private final Waiter waiter;
        private final CourseType courseType;
        private Course announcedCourse;

        public WaiterAnnounceCourseThread(Waiter waiter, CourseType courseType) {
            this.waiter = waiter;
            this.courseType = courseType;
        }

        @Override
        public void run() {
            try {
                announcedCourse = waiter.announceCourse(courseType);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }

        public Course getAnnouncedCourse() {
            return announcedCourse;
        }
    }

    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException, OutOfStockException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        var starterThread = new WaiterAnnounceCourseThread(grover, CourseType.STARTER);
        var mainThread = new WaiterAnnounceCourseThread(zoe, CourseType.MAIN);
        var dessertThread = new WaiterAnnounceCourseThread(rosita, CourseType.DESSERT);

        starterThread.start();
        mainThread.start();
        dessertThread.start();

        starterThread.join();
        mainThread.join();
        dessertThread.join();

        return new MultiCourseMeal(starterThread.getAnnouncedCourse(),
                                   mainThread.getAnnouncedCourse(),
                                   dessertThread.getAnnouncedCourse());
    }
}
```

## RUN: MainRestaurant

# Demo 3 - MultiWaiterExecutorServiceRestaurant

*(tag `2-deep-dive-created-multi-waiter-threads-restaurant`)*

## CREATE: MultiWaiterExecutorServiceRestaurant

```java
public class MultiWaiterExecutorServiceRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newFixedThreadPool(3)) {
            Future<Course> starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future<Course> main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future<Course> dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
```

## RUN: MainRestaurant

# Demo 4 - AnnouncementId

*(tag `3-deep-dive-created-multi-waiter-executor-service-restaurant`)*

## CREATE: `AnnouncementId`

```java
package com.github.hannotify.structuredconcurrency.restaurant.announcement;

import java.util.concurrent.atomic.AtomicInteger;

public class AnnouncementId {
    private static final AtomicInteger nextId = new AtomicInteger(1);
    private static final ThreadLocal<Integer> announcementId = ThreadLocal.withInitial(nextId::getAndIncrement);

    public static int get() {
        return announcementId.get();
    }
}
```

## CALL: `Waiter`, `Chef`

When printing announcements to System.out.

## RUN: `MainRestaurant`

# Demo 5 - ShutdownOnFailure

*(tag `4-deep-dive-generated-announcement-ids)`)*

## CREATE: `StructuredConcurrencyRestaurant`

```java
package com.github.hannotify.structuredconcurrency.restaurant;

import com.github.hannotify.structuredconcurrency.restaurant.kitchen.CourseType;
import com.github.hannotify.structuredconcurrency.restaurant.kitchen.MultiCourseMeal;
import com.github.hannotify.structuredconcurrency.staff.Waiter;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.StructuredTaskScope;

public class StructuredConcurrencyRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        // waiters
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            // announceCourses
            var starter = scope.fork(() -> grover.announceCourse(CourseType.STARTER));
            var main = scope.fork(() -> zoe.announceCourse(CourseType.MAIN));
            var dessert = scope.fork(() -> rosita.announceCourse(CourseType.DESSERT));

            scope.join().throwIfFailed();

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
```

## EXPLAIN join(), throwIfFailed(), get(): 

`join()` blocks, `throwIfFailed` optionally throws, `get()` always returns a valid result

## EXPLAIN Subtask: 

the introduction of `Subtask` (meant for calling 'get()'s after a result is already known, unlike (Completable)Future)

## RUN: `MainRestaurant`

# Demo 6 - ShutdownOnSuccess

*(tag `5-deep-dive-created-structured-concurrency-restaurant`)*

## SHOW: `Waiter.getDrinkOrder()`

## SHOW: `Guest`

## CREATE: `MainBar`

```java
void main() throws Exception {
    Bar bar = new MultiWaiterInvokeAnyBar();

    Guest hanno = new Guest("Hanno", List.of(
            new Drink("Espresso", DrinkCategory.COFFEE),
            new Drink("Westmalle Dubbel", DrinkCategory.BEER)
    ));

    Guest rianne = new Guest("Rianne", List.of(
            new Drink("Cappuccino", DrinkCategory.COFFEE),
            new Drink("Green tea", DrinkCategory.TEA)
    ));

    bar.determineDrinkOrder(hanno);
    bar.determineDrinkOrder(rianne);
}
```
## CREATE: `StructuredConcurrencyBar`

```java
public class StructuredConcurrencyBar implements Bar {
    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws Exception {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<DrinkOrder>()) {
            scope.fork(() -> zoe.getDrinkOrder(guest, COFFEE, BEER, SOFT_DRINK));
            scope.fork(() -> elmo.getDrinkOrder(guest, DISTILLED, WINE, COCKTAIL, TEA));

            return scope.join().result();
        }
    }
}
```
## EXPLAIN: `join()` blocks, again.

## EXPLAIN: `result()` returns the result of the first subtask that completed.

## RUN: `MainBar`

## EXPLAIN: the behavior

# Demo 7 (optional) - MultiWaiterInvokeAll

*(tag `6-deep-dive-created-structured-concurrency-bar`)*

## CREATE: `MultiWaiterInvokeAllRestaurant`

```java
public class MultiWaiterInvokeAllRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            List<Future<Course>> futures = executor.invokeAll(List.of(
                    () -> grover.announceCourse(CourseType.STARTER),
                    () -> zoe.announceCourse(CourseType.MAIN),
                    () -> rosita.announceCourse(CourseType.DESSERT)
            ));

            return new MultiCourseMeal(futures.stream().map(f -> {
                try {
                    return f.get();
                } catch (InterruptedException | ExecutionException e) {
                    throw new RuntimeException(e);
                }
            }).toArray(Course[]::new));
        }
    }
}
```

## RUN: `MainRestaurant`

## CHANGE: 

The following line in both `StructuredConcurrencyRestaurant` and `MultiWaiterInvokeAllRestaurant`:

```java
var starter = scope.fork(() -> { throw new RuntimeException("sorry, no starters today"); });
```

## SHOW: 

How `StructuredConcurrencyRestaurant` supports cancellation, and how `ExecutorService.invokeAll()` actually doesn't.

# Demo 8 - ScopedValue

*(tag `7-deep-dive-created-multi-waiter-invoke-all-restaurant-demonstrated-cancellation`)*

## CHANGE: `AnnouncementId` to have a ScopedValue instead of a ThreadLocal

```java
public class AnnouncementId {
    private static final AtomicInteger nextId = new AtomicInteger(1);
    private static final ScopedValue<Integer> scopedValue = ScopedValue.newInstance();

    public static int nextId() {
        return nextId.getAndIncrement();
    }

    public static ScopedValue<Integer> scopedValue() {
        return scopedValue;
    }

    public static int get() {
        return scopedValue.get();
    }
}
```

## CALL: 

`ScopedValue.where(..)` in `Waiter.announceCourse(courseType)`.

## CALL: 

`ScopedValue.get()` outside of the scope to show what happens.

## EXPLAIN: how scope works

We see that scoped values can only be accessed within the defined scope, leading to a limited lifetime ('how long is it accessible?') and access ('by who is it accessible?').
