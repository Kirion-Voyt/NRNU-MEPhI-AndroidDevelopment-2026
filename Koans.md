# 	Kotlin Koans

## Introduction
### Hello, world!
``` kotlin
fun start(): String = "OK"
```

### Named arguments
``` kotlin
fun joinOptions(options: Collection<String>) =
 options.joinToString(", ", "[", "]")
```

### Default arguments
``` kotlin
fun foo(name: String, number: Int = 42, toUpperCase: Boolean = false) =
	(if (toUpperCase) name.uppercase() else name) + number

fun useFoo() = listOf(
	foo("a"),
	foo("b", number = 1),
	foo("c", toUpperCase = true),
	foo(name = "d", number = 2, toUpperCase = true)
)
```

### Triple-quoted strings
``` kotlin
const val question = "life, the universe, and everything"
const val answer = 42

val tripleQuotedString = """
	#question = "$question"
	#answer = $answer""".trimMargin("#")
    
fun main() {
	println(tripleQuotedString)
}
```

### String templates
``` kotlin
val month = "(JAN|FEB|MAR|APR|MAY|JUN|JUL|AUG|SEP|OCT|NOV|DEC)"

fun getPattern(): String = """\d{2} $month \d{4}"""
```

### Nullable types
``` kotlin
fun sendMessageToClient(
    client: Client?, message: String?, mailer: Mailer
) {
	val email: String? = client?.personalInfo?.email;

	if (client == null || message == null || email == null) return;
	mailer.sendMessage(email, message);
}

class Client(val personalInfo: PersonalInfo?)
class PersonalInfo(val email: String?)

interface Mailer {
	fun sendMessage(email: String, message: String)
}
```

### Nothing type
``` kotlin
import java.lang.IllegalArgumentException

fun failWithWrongAge(age: Int?): Nothing {
	throw IllegalArgumentException("Wrong age: $age")
}

fun checkAge(age: Int?) {
	if (age == null || age !in 0..150) failWithWrongAge(age)
	println("Congrats! Next year you'll be ${age + 1}.")
}

fun main() {
	checkAge(10)
}
```

### Lambdas
``` kotlin
fun containsEven(collection: Collection<Int>): Boolean =
	collection.any { it % 2 == 0 }
```

## Classes
### Data classes
``` kotlin
data class Person(val name: String, val age: Int)

fun getPeople(): List<Person> {
    return listOf(Person("Alice", 29), Person("Bob", 31))
}

fun comparePeople(): Boolean {
    val p1 = Person("Alice", 29)
    val p2 = Person("Alice", 29)
    return p1 == p2  // should be true
}
```

### Smart casts
``` kotlin
fun eval(expr: Expr): Int =
	when (expr) {
		is Num -> return expr.value;
		is Sum -> return eval(expr.left) + eval(expr.right);
		else -> throw IllegalArgumentException("Unknown expression");
	}

interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```

### Sealed classes
``` kotlin
fun eval(expr: Expr): Int =
	when (expr) {
		is Num -> return expr.value;
		is Sum -> return eval(expr.left) + eval(expr.right);
	}
    
sealed interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```

### Rename on import
``` kotlin
import kotlin.random.Random as KRandom
import java.util.Random as JRandom

fun useDifferentRandomClasses(): String {
	return "Kotlin random: " +
	KRandom.nextInt(2) +
	" Java random:" +
	JRandom().nextInt(2) +
	"."
}
```

### Extension functions
``` kotlin
fun Int.r(): RationalNumber = RationalNumber(this, 1);

fun Pair<Int, Int>.r(): RationalNumber = 
    RationalNumber(this.first, this.second);

data class RationalNumber(val numerator: Int, val denominator: Int)
```

## Conventions
### Comparison
``` kotlin
data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int) :
	Comparable<MyDate> {
	override fun compareTo(other : MyDate) : Int {
		if (this.dayOfMonth == other.dayOfMonth) {
			if (this.month == other.month) {
				if (this.year == other.year) {
					return 0;
				}

 			return (this.year - other.year);
 			}
		return (this.month - other.month);
		}
	return (this.dayOfMonth - other.dayOfMonth);
	}
}

fun test(date1: MyDate, date2: MyDate) {
    // this code should compile:
    println(date1 < date2)
}
```

### Ranges
``` kotlin
fun checkInRange(date: MyDate, first: MyDate, last: MyDate): Boolean {
	return date in first..last;
}
```

### For loop
``` kotlin
class DateRange(val start: MyDate, val end: MyDate) : Iterable<MyDate> {
	override fun iterator(): Iterator<MyDate> = object : Iterator<MyDate> {
		var current: MyDate = start
		var hasNext: Boolean = current <= end
		override fun hasNext(): Boolean = hasNext
		override fun next(): MyDate {
			if (!hasNext) throw NoSuchElementException()
			val result = current
			hasNext = current < end
			current = current.followingDate()
			return result
		}
	}
}

fun iterateOverDateRange(firstDate: MyDate, secondDate: MyDate, handler: (MyDate) -> Unit) {
    for (date in firstDate..secondDate) {
        handler(date)
    }
}
```

### Operators overloading
``` kotlin
import TimeInterval.*
data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int)
// Supported intervals that might be added to dates:

enum class TimeInterval { DAY, WEEK, YEAR }
class intervalRep(val interval : TimeInterval, val rep : Int)

operator fun TimeInterval.times(num : Int) = intervalRep(this, num)
operator fun MyDate.plus(timeInterval: TimeInterval): MyDate =
	addTimeIntervals(timeInterval, 1)
operator fun MyDate.plus(timeInterval: intervalRep) : MyDate =
	addTimeIntervals(timeInterval.interval, timeInterval.rep)

fun task1(today: MyDate): MyDate {
    return today + YEAR + WEEK
}

fun task2(today: MyDate): MyDate {
    return today + YEAR * 2 + WEEK * 3 + DAY * 5
}
```

### Invoke
``` kotlin
class Invokable {
	var numberOfInvocations: Int = 0
	private set
	operator fun invoke(): Invokable {
		numberOfInvocations++
		return this
	}
}

fun invokeTwice(invokable: Invokable) = invokable()()
```

## Collections
### Introduction
``` kotlin
fun Shop.getSetOfCustomers(): Set<Customer> =
        customers.toSet()
```

### Sort
``` kotlin
// Return a list of customers, sorted in the descending by number of orders they have made
fun Shop.getCustomersSortedByOrders(): List<Customer> =
        customers.sortedByDescending { it.orders.size }
```

### Filter map
``` kotlin
// Find all the different cities the customers are from
fun Shop.getCustomerCities(): Set<City> =
        customers.map { it.city }.toSet()

// Find the customers living in a given city
fun Shop.getCustomersFrom(city: City): List<Customer> =
        customers.filter { it.city == city }
```

### All Any and other predicates
``` kotlin
// Return true if all customers are from a given city
fun Shop.checkAllCustomersAreFrom(city: City): Boolean =
        customers.all { it.city == city }

// Return true if there is at least one customer from a given city
fun Shop.hasCustomerFrom(city: City): Boolean =
        customers.any { it.city == city }

// Return the number of customers from a given city
fun Shop.countCustomersFrom(city: City): Int =
        customers.count { it.city == city }

// Return a customer who lives in a given city, or null if there is none
fun Shop.findCustomerFrom(city: City): Customer? =
        customers.find { it.city == city }
```

### Associate
``` kotlin
// Build a map from the customer name to the customer
fun Shop.nameToCustomerMap(): Map<String, Customer> =
        customers.associateBy { it.name }

// Build a map from the customer to their city
fun Shop.customerToCityMap(): Map<Customer, City> =
        customers.associateWith { it.city }

// Build a map from the customer name to their city
fun Shop.customerNameToCityMap(): Map<String, City> =
        customers.associate { it.name to it.city }
```

### GroupBy
``` kotlin
// Build a map that stores the customers living in a given city
fun Shop.groupCustomersByCity(): Map<City, List<Customer>> =
        customers.groupBy { it.city }
```

### Partition
``` kotlin
// Return customers who have more undelivered orders than delivered
fun Shop.getCustomersWithMoreUndeliveredOrders(): Set<Customer> = 
    customers.filter { 
        val (d, u) = it.orders.partition { it.isDelivered }
            u.size > d.size }.toSet()
```

### FlatMap
``` kotlin
// Return all products the given customer has ordered
fun Customer.getOrderedProducts(): List<Product> =
        orders.flatMap(Order::products)

// Return all products that were ordered by at least one customer
fun Shop.getOrderedProducts(): Set<Product> =
        customers.flatMap(Customer::getOrderedProducts).toSet()
```

### Max min
``` kotlin
// Return a customer who has placed the maximum amount of orders
fun Shop.getCustomerWithMaxOrders(): Customer? =
        customers.maxByOrNull { it.orders.size }

// Return the most expensive product that has been ordered by the given customer
fun getMostExpensiveProductBy(customer: Customer): Product? =
        customer.orders.flatMap(
            Order::products
        ).maxByOrNull(
            Product::price
        )
```

### Sum
``` kotlin
// Return the sum of prices for all the products ordered by a given customer
fun moneySpentBy(customer: Customer): Double =
        customer.orders.flatMap(Order::products).sumOf { it.price }
```

###Fold and reduce
``` kotlin
// Return the set of products that were ordered by all customers
fun Shop.getProductsOrderedByAll(): Set<Product> =
    customers.map(Customer::getOrderedProducts).reduce {
        orderedByAll, customer ->
        orderedByAll.intersect(customer)
    }
    
fun Customer.getOrderedProducts(): Set<Product> =
    orders.flatMap(Order::products).toSet()
```

### Compound tasks
``` kotlin
// Find the most expensive product among all the delivered products
// ordered by the customer. Use `Order.isDelivered` flag.
fun findMostExpensiveProductBy(customer: Customer): Product? =
    customer.orders.filter(Order::isDelivered).flatMap(
            Order::products
        ).maxByOrNull(Product::price)

// Count the amount of times a product was ordered.
// Note that a customer may order the same product several times.
fun Shop.getNumberOfTimesProductWasOrdered(product: Product): Int =
    customers.flatMap(Customer::getOrderedProducts).count { it == product }

fun Customer.getOrderedProducts(): List<Product> =
        orders.flatMap(Order::products)
```

### Sequences
``` kotlin
// Find the most expensive product among all the delivered products
// ordered by the customer. Use `Order.isDelivered` flag.
fun findMostExpensiveProductBy(customer: Customer): Product? =
    customer.orders.asSequence().filter(Order::isDelivered)
        .flatMap(Order::products).maxByOrNull(Product::price)

// Count the amount of times a product was ordered.
// Note that a customer may order the same product several times.
fun Shop.getNumberOfTimesProductWasOrdered(product: Product): Int =
    customers.asSequence().flatMap(Customer::getOrderedProducts)
        .count { it == product }

fun Customer.getOrderedProducts(): Sequence<Product> =
        orders.asSequence().flatMap(Order::products)
```

### Getting used to new style
``` kotlin
fun doSomethingWithCollection(collection: Collection<String>): Collection<String>? {

    val groupsByLength = collection.groupBy { s -> s.length }

    val maximumSizeOfGroup = groupsByLength.values
        .map { group -> group.size }.maxOrNull()

    return groupsByLength.values
        .firstOrNull { group -> group.size == maximumSizeOfGroup}
}
```

## Properties
### Properties
``` kotlin
class PropertyExample() {
    var counter = 0
    var propertyWithCounter: Int? = null
        set(v) {
            field = v
            counter++
        }
}
```

### Lazy property
``` kotlin
class LazyProperty(val initializer: () -> Int) {
    var value: Int? = null
    val lazy: Int
        get() {
            if (value == null) { value = initializer() }
            return value!!
        }
}
```

### Delegates examples
``` kotlin
import kotlin.reflect.KProperty

class LazyProperty(val initializer: () -> Int) {
    val lazyValue: Int by lazy { initializer() }
}
```

### Delegates how it works
``` kotlin
import kotlin.properties.ReadWriteProperty
import kotlin.reflect.KProperty

class D {
    var date: MyDate by EffectiveDate()
}

class EffectiveDate<R> : ReadWriteProperty<R, MyDate> {

    var timeInMillis: Long? = null

    override fun getValue(thisRef: R, property: KProperty<*>): MyDate {
        return timeInMillis!!.toDate()
    }

    override fun setValue(thisRef: R, property: KProperty<*>, value: MyDate) {
       	timeInMillis = value.toMillis()
    }
}
```
