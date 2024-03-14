# Feedback For Friend's Project

[wolt\_project](https://github.com/jensjvh/wolt\_project)

**High-Level Potential Improvements**

* [**CI/CD pipelines**](https://www.redhat.com/en/topics/devops/what-cicd-pipeline)
  * For example, [Renovate Bot](https://docs.renovatebot.com/python/) integration.
  * The tests in _test\_api.py_ could have been implemented to automatically run in a CI/CD pipeline with a _test_ stage. Overall, [GitOps](https://about.gitlab.com/topics/gitops/) could've been configured.
    * This is also known as [Automated Testing](https://www.atlassian.com/continuous-delivery/software-testing/automated-testing).
    * Book recommendation: [The DevOps Handbook](https://www.amazon.com/DevOps-Handbook-World-Class-Reliability-Organizations/dp/1942788002).
* **Pytest**
  * A good _Testing Framework_, but would it be beneficial to use a different one?
    * For example, [Robot Framework](https://robotframework.org/) (RF) is pretty widely known and used across Finland. RF also originates from Finland (possible benefits of bias?)
    * RF could've _potentially_ made the written tests more `readable` and `understandable` as it uses [Keyword-Driven Testing](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html).
      * **^This is purely speculation**.
* [**OpenAPI Specification (OAS)**](https://swagger.io/specification/)
  * Could have been used when documenting the Python REST API.
    * _OpenAPI_ is the _de-facto_, programming language-agnostic standard for REST API documentation.

**Low-Level Potential Code Improvements**

* https://github.com/jensjvh/wolt\_project/blob/main/api.py#L36
  * That source line of code ([SLOC](https://en.wikipedia.org/wiki/Source\_lines\_of\_code)) could have been made more _readable_.
  * Validation could have been moved into a more cohesive and [decoupled](https://softwareengineering.stackexchange.com/questions/244476/what-is-decoupling-and-what-development-areas-can-it-apply-to) place (e.g., its own class or into a reusable function).
    * Additionally, more specific validation could've been implemented
  * Error handling is too abstract and could be more specific and helpful.
    * For example, if `values['cartvalue']` is invalid the user will be met with an abstract error message `error: invalid request format`instead of an explicit `error: cart value should be a float/integer`.
* You could familiarize yourself with [DTOs](https://dev.to/izabelakowal/some-ideas-on-how-to-implement-dtos-in-python-be3), [Service Layer Pattern](https://en.wikipedia.org/wiki/Service\_layer\_pattern), and [MVC with Python](https://realpython.com/the-model-view-controller-mvc-paradigm-summarized-with-legos/).
* [Automatic linter](https://code.visualstudio.com/docs/python/linting) could've been configured for collaborative development and readability
  * _You maybe had this already(?)_
* https://github.com/jensjvh/wolt\_project/blob/main/fee.py#L98
  * Naming of conditionals (i.e., giving conditional names)
    * This can make the code more readable
    * [This](https://medium.com/readable-code/readable-code-how-to-write-readable-conditional-statements-626cb17a1d2a) article is very informative and concrete.
    * For example, consider this refactor: `is_delivery_free = self._cart_value >= DeliveryFee.max_cart_value`
* Method improvements:
  * Some methods could be private in [fee.py](https://github.com/jensjvh/wolt\_project/blob/main/fee.py).
    * All the functions starting with `check_` should be private because they aren't supposed to be called outside of the class (as they're currently only called from inside the class itself). [More information here](https://stackoverflow.com/questions/15047122/python-private-function-coding-convention).
    * Additionally, all the public methods could be moved to the top (above the private methods).
      * It's nice for Public Methods to have more visibility than private ones.
  * Consider naming Methods more descriptively so that they describe what they do.
    * For example, `check_cart_value` not only checks the value but _changes_ it. This causes side effects, which compound on the rows [102-104](https://github.com/jensjvh/wolt\_project/blob/main/fee.py#L102)
    * [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single\_responsibility\_principle) (SRP) is a good technique for making functions/methods more valid and fault tolerant as they should do ONE thing only and do that thing well.
  * Comments could have been reduced
    * One of the most powerful ways to reduce comments is by making the code more straightforward and readable. This way you're kind of "integrating documentation into the code" without needing any comments at all.
      * [Tip 13](https://pragprog.com/tips/) and pages 23 and 34 from [The Pragmatic Programmer](https://www.amazon.com/Pragmatic-Programmer-journey-mastery-Anniversary/dp/0135957052) book.
  * Void methods (i.e., methods that don't return values) should be contemplated thoroughly before implementing them as [they can be considered to be anti-patterns](https://dzone.com/articles/void-methods-considered-anti-pattern).
    * Your code uses a lot of [Void Methods](https://www.geeksforgeeks.org/difference-between-void-and-non-void-methods-in-java/) (e.g., all the `check_` methods), which have side effects.
    * Consider using [Pure Functions/Methods](https://www.studysmarter.co.uk/explanations/computer-science/functional-programming/pure-function/) . They are deterministic, idempotent, and they do not cause [side effects](https://reintech.io/terms/category/side-effects-in-software-development).
      * Pure functions are part of the [Functional Programming](https://www.geeksforgeeks.org/functional-programming-paradigm/) paradigm.
  * Recommended reading: [Summary of 'Clean Code' by Robert C. Martin.](https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29) and [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882).

**One Possible Way Of Implementing `DeliveryFee` and `Fee` Class Implementations with Pure Methods and Less Side-Effects**

1. `class.py`
   * Removed mutable class instance variables
   * Renamed & repurposed methods
   * No void methods
2. `static.py`
   * Static + pure methods
   * No void methods

**These examples are untested and should only be used as learning/reference material.**

**`class.py`**

```python
from math import ceil
import datetime

class DeliveryFee():

    #Constants as class attributes
    max_cart_value = 20000
    max_total_fee = 1500
    base_delivery_cost = 200
    base_delivery_distance = 1000
    
    def __init__(self, cart_value: int, distance: int, no_of_items: int, time: str):
        self._cart_value = cart_value
        self._distance = distance
        self._no_of_items = no_of_items
        self._time = time

    def calculate_delivery_fee(self):
        grant_free_delivery = self.cart_value >= DeliveryFee.max_cart_value
        if grant_free_delivery:
            return 0 
        
        surcharge = self.__calculate_surcharge()
        delivery_cost = self.__calculate_delivery_cost()
        multiplier = self.__get_fee_multiplier()

        total = (delivery_cost + surcharge) * multiplier

        return min(DeliveryFee.max_total_fee, total)

    def __calculate_surcharge(self):
        surcharge = 0
        if self._cart_value < 1000:
            surcharge += abs(1000 - self._cart_value)
        
        if self._item_count > 5: 
            surcharge += (self._item_count - 5) * 50
        
        if self._item_count > 12:
            surcharge += 120

        return surcharge

    def __calculate_delivery_cost(distance: int):
        cost = DeliveryFee.base_delivery_cost

        if distance > DeliveryFee.base_delivery_distance:
            remaining = distance - DeliveryFee.base_delivery_distance
            cost += ceil(remaining / 500) * 100

        return cost

    def __get_fee_multiplier(self):
        datetime_obj = datetime.datetime.fromisoformat(self._time)
        
        if self.__is_rush_hour_day(datetime_obj) and self.__is_rush_hour_time(datetime_obj):
            return 1.2
        return 1

    def __is_rush_hour_day(datetime_obj: datetime.datetime):
        friday = 4
        return datetime_obj.weekday() == friday
    
    def __is_rush_hour_time(datetime_obj: datetime.datetime):
        rush_start, rush_end = datetime.time(15, 0, 0), datetime.time(19, 0, 0)
        current_time = datetime_obj.time()
        return current_time >= rush_start and current_time < rush_end
```

**`static.py`**

```python
from math import ceil
import datetime

BASE_DELIVERY_COST = 200
BASE_DELIVERY_DISTANCE = 1000
MAX_CART_VALUE = 20000

class Fee: 

    # Public
    @staticmethod
    def calculateDelivery(cart_value: int, distance: int, item_count: int, time: str):
        grant_free_delivery = cart_value >= Fee.max_cart_value
        if grant_free_delivery:
            return 0

        surcharge = Fee.__calculate_surcharge(cart_value, item_count)
        delivery_cost = Fee.__calculate_delivery_cost(distance)
        multiplier = Fee.__get_fee_multiplier(time)
        
        return (surcharge + delivery_cost) * multiplier


    @staticmethod
    def __calculate_surcharge(cart_value: int, item_count: int):
        surcharge = 0
        if cart_value < 1000:
            surcharge += abs(1000 - cart_value)
        
        if item_count > 5: 
            surcharge += (item_count - 5) * 50
        
        if item_count > 12:
            surcharge += 120

        return surcharge


    @staticmethod
    def __calculate_delivery_cost(distance: int):
        cost = BASE_DELIVERY_COST

        if distance > BASE_DELIVERY_DISTANCE:
            remaining = distance - BASE_DELIVERY_DISTANCE
            cost += ceil(remaining / 500) * 100

        return cost


    @staticmethod
    def __get_fee_multiplier(time: str):
        datetime_obj = datetime.datetime.fromisoformat(time)
        
        if Fee.__is_rush_hour_day(datetime_obj) and Fee.__is_rush_hour_time(datetime_obj):
            return 1.2
        return 1


    @staticmethod
    def __is_rush_hour_day(datetime_obj: datetime.datetime):
        friday = 4
        return datetime_obj.weekday() == friday
    

    @staticmethod
    def __is_rush_hour_time(datetime_obj: datetime.datetime):
        rush_start, rush_end = datetime.time(15, 0, 0), datetime.time(19, 0, 0)
        current_time = datetime_obj.time()
        return current_time >= rush_start and current_time < rush_end
```
