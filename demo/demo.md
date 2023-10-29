```
package com.github.hannotify.structuredconcurrency.restaurant.announcement;

import java.util.concurrent.atomic.AtomicInteger;

public class AnnouncementId {
    private static final AtomicInteger nextId = new AtomicInteger(1);
    private static final ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(AnnouncementId::nextId);
    private static final ScopedValue<Integer> scopedValue = ScopedValue.newInstance();

    public static int nextId() {
        return nextId.getAndIncrement();
    }

    public static ThreadLocal<Integer> threadLocal() {
        return threadLocal;
    }

    public static ScopedValue<Integer> scopedValue() {
        return scopedValue;
    }
}
```

```
package com.github.hannotify.structuredconcurrency.restaurant;

import com.github.hannotify.structuredconcurrency.restaurant.kitchen.CourseType;
import com.github.hannotify.structuredconcurrency.restaurant.kitchen.MultiCourseMeal;
import com.github.hannotify.structuredconcurrency.staff.Waiter;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.StructuredTaskScope;

public class StructuredConcurrencyRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            var starter = scope.fork(() -> grover.announceCourse(CourseType.STARTER));
            var main = scope.fork(() -> zoe.announceCourse(CourseType.MAIN));
            var dessert = scope.fork(() -> rosita.announceCourse(CourseType.DESSERT));

            scope.join().throwIfFailed();

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
```

```
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