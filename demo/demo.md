# Demo 1 - SingleWaiterRestaurant

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

# Demo 2 - MultiWaiterThreadsRestaurant

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

# Demo 3 - MultiWaiterExecutorServiceRestaurant

```java
public class MultiWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            Future<Course> starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future<Course> main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future<Course> dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
```

# Demo 4 - AnnouncementId

```java
package com.github.hannotify.structuredconcurrency.restaurant.announcement;

import java.util.concurrent.atomic.AtomicInteger;

public class AnnouncementId {
    private static final AtomicInteger nextId = new AtomicInteger(1);
    private static final ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(AnnouncementId::nextId);

    public static int nextId() {
        return nextId.getAndIncrement();
    }

    public static ThreadLocal<Integer> threadLocal() {
        return threadLocal;
    }
}
```

# Demo 5 - ShutdownOnFailure

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

# Demo 6 - ShutdownOnSuccess

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

# Demo 7 - MultiWaiterInvokeAll

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

# Demo 8 - ScopedValue

```java
package com.github.hannotify.structuredconcurrency.restaurant.announcement;

import java.util.concurrent.atomic.AtomicInteger;

public class AnnouncementId {
    private static final AtomicInteger nextId = new AtomicInteger(1);
    private static final ScopedValue<Integer> scopedValue = ScopedValue.newInstance();

    public static int nextId() {
        return nextId.getAndIncrement();
    }

    public static ScopedValue<Integer> scopedValue() {
        return scopedValue;
    }
}
```

```java
package com.github.hannotify.structuredconcurrency.bar;

import com.github.hannotify.structuredconcurrency.staff.Waiter;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.StructuredTaskScope;

import static com.github.hannotify.structuredconcurrency.bar.DrinkCategory.*;

public class StructuredConcurrencyBar implements Bar {
    private static final ScopedValue<Integer> drinkOrderId = ScopedValue.newInstance();

    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws Exception {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        return ScopedValue.where(drinkOrderId, 1)
                .call(() -> {
                    try (var scope = new StructuredTaskScope.ShutdownOnSuccess<DrinkOrder>()) {
                        scope.fork(() -> zoe.getDrinkOrder(guest, BEER, WINE, JUICE));
                        scope.fork(() -> elmo.getDrinkOrder(guest, COFFEE, TEA, COCKTAIL, DISTILLED));

                        return scope.join().result();
                    }
                });
    }
}
```