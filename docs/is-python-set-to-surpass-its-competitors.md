# Python 是否即将超越其竞争对手？

> 原文：[`towardsdatascience.com/is-python-set-to-surpass-its-competitors/`](https://towardsdatascience.com/is-python-set-to-surpass-its-competitors/)

松露是一种起源于 18 世纪的法国烤蛋菜肴。制作一个优雅美味的法国松露的过程非常复杂，在过去，这通常只由专业的法国糕点师准备。然而，随着超市里现在广泛可用的预制松露混合物的出现，这道经典的法国菜肴已经进入了无数家庭的厨房。

Python 就像编程中的预制松露混合物。许多研究一致表明，Python 是开发者中最受欢迎的编程语言，这种优势将在 2025 年继续扩大。与 C、C++、Java 和 Julia 等语言相比，Python 脱颖而出，因为它高度可读和表达性强，灵活且动态，易于初学者使用且功能强大。这些特性使 Python 成为连编程基础都没有的人最合适的编程语言。以下特性使 Python 区别于其他编程语言：

+   动态类型

+   列表推导式

+   生成器

+   参数传递和可变性

这些特性揭示了 Python 作为编程语言的内在本质。没有这些知识，你永远无法真正理解 Python。在今天的文章中，我将详细阐述 Python 如何通过这些特性在其它编程语言中脱颖而出。

## 动态类型

对于大多数编程语言，如 Java 或 C++，需要显式声明数据类型。但到了 Python，当你创建一个变量时，不需要声明其类型。Python 的这个特性称为动态类型，这使得 Python 灵活且易于使用。

## 列表推导式

列表推导式用于通过将函数应用于列表中的每个元素来从其他列表生成列表。它们提供了一种简洁的方式来在列表中应用循环和可选条件。

例如，如果你想创建一个包含 0 到 9 之间偶数的平方的列表，你可以使用 JavaScript、Python 中的常规循环和 Python 的列表推导式来实现相同的目标。

### JavaScript

```py
let squares = Array.from({ length: 10 }, (_, x) => x)  // Create array [0, 1, 2, ..., 9]
   .filter(x => x % 2 === 0)                          // Filter even numbers
   .map(x => x ** 2);                                 // Square each number
console.log(squares);  // Output: [0, 4, 16, 36, 64]
```

### Python 中的常规循环

```py
squares = []
for x in range(10):
   if x % 2 == 0:
       squares.append(x**2)
print(squares) 
```

### Python 的列表推导式

```py
squares = [x**2 for x in range(10) if x % 2 == 0]print(squares) 
```

上面的三个代码部分都生成了相同的列表[0, 4, 16, 36, 64]，但 Python 的列表推导式最为优雅，因为其语法简洁且清晰地表达了意图，而 Python 函数则更为冗长，需要显式初始化和追加。JavaScript 的语法最不优雅且不易读，因为它需要链式使用 Array.from、filter 和 map 方法。Python 函数和 JavaScript 函数都不直观，不能像 Python 列表推导式那样自然地阅读。

## 生成器

Python 中的生成器是一种特殊的迭代器，允许开发者迭代一系列值，而无需一次性将它们全部存储在内存中。它们通过 yield 关键字创建。尽管像 C++ 和 Java 这样的其他编程语言提供了类似的功能，但它们没有以相同简单、集成的方式内置 yield 关键字。以下是一些使 Python 生成器独特的关键优势：

+   **内存效率**：生成器一次只产生一个值，因此它们在任何给定时刻只计算并保持一个项目在内存中。这与 Python 中的列表形成对比，列表在内存中存储所有项目。

+   **延迟评估**：生成器使 Python 能够仅在需要时计算值。这种“懒惰”的计算在处理大型或可能无限长的序列时会导致显著的性能提升。

+   **简单语法**：这可能是开发者选择使用生成器的最大原因，因为他们可以轻松地将常规函数转换为生成器，而无需显式管理状态。

```py
def fibonacci():
   a, b = 0, 1
   while True:
       yield a
       a, b = b, a + b

fib = fibonacci()
for _ in range(100):
   print(next(fib))
```

上面的示例展示了在创建序列时如何使用 yield 关键字。对于有生成器和没有生成器的代码在内存使用和时间上的差异，生成 100 个斐波那契数几乎看不到任何区别。但在实际操作中，当涉及到 1 亿个数字时，你最好使用生成器，因为 1 亿个数字的列表很容易耗尽许多系统资源。

## 参数传递和可变性

在 Python 中，我们实际上并没有给变量赋值；相反，我们绑定变量到对象上。这种操作的结果取决于对象是否可变。如果一个对象是可变的，那么在函数内部对其所做的更改将影响原始对象。

```py
def modify_list(lst):
   lst.append(4)

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)  # Output: [1, 2, 3, 4]
```

在上面的示例中，我们想要将数字 '4' 添加到列表 my_list 中，该列表为 [1,2,3]。因为列表是可变的，所以 append 操作的行为会改变原始列表 my_list，而不会创建一个副本。

然而，不可变对象，如整数、浮点数、字符串、元组和冻结集合，在创建后不能被更改。因此，任何修改都会导致创建一个新的对象。在下面的示例中，因为整数是不可变的，所以函数创建了一个新的整数而不是修改原始变量。

```py
def modify_number(n):
   n += 10
   return n

a = 5
new_a = modify_number(a)
print(a)      # Output: 5
print(new_a)  # Output: 15
```

Python 的参数传递有时被描述为“按对象引用传递”或“按赋值传递”。这使得 Python 独特，因为 Python 统一地传递引用（按对象引用传递），而其他语言需要明确区分按值传递和按引用传递。Python 的统一方法简单而强大。它避免了显式指针或引用参数的需要，但要求开发者注意可变对象。

在使用 Python 的参数传递和可变性时，我们可以在编码中享受到以下好处：

+   **内存效率**：通过传递引用而不是制作对象的完整副本来节省内存。这特别有利于使用大型数据结构的代码开发。

+   **性能:** 它避免了不必要的复制，从而提高了整体的编码性能。

+   **灵活性:** 这个特性为更新数据结构提供了便利，因为开发者不需要明确选择按值传递和按引用传递。

然而，Python 的这个特性迫使开发者必须仔细选择可变和不可变数据类型，这也带来了更复杂的调试问题。

## 那 Python 真的简单吗？

Python 的流行源于其简单性、内存效率、高性能和易用性。它也是一种看起来最像人类自然语言的编程语言，因此即使没有接受过系统性和全面的编程培训的人也能理解它。这些特性使 Python 成为企业、学术机构和政府组织的热门选择。

例如，当我们想过滤出金额大于 200 的“已完成”订单，并更新一个可变的总结报告（一个字典），包含电子商务公司的总计数和金额总和时，我们可以使用***列表推导式***创建一个符合我们标准的订单列表，***跳过变量类型的声明***，并通过***按赋值传递***修改原始字典。

```py
import random
import time

def order_stream(num_orders):
   """
   A generator that yields a stream of orders.
   Each order is a dictionary with dynamic types:
     - 'order_id': str
     - 'amount': float
     - 'status': str (randomly chosen among 'completed', 'pending', 'cancelled')
   """
   for i in range(num_orders):
       order = {
           "order_id": f"ORD{i+1}",
           "amount": round(random.uniform(10.0, 500.0), 2),
           "status": random.choice(["completed", "pending", "cancelled"])
       }
       yield order
       time.sleep(0.001)  # simulate delay

def update_summary(report, orders):
   """
   Updates the mutable summary report dictionary in-place.
   For each order in the list, it increments the count and adds the order's amount.
   """
   for order in orders:
       report["count"] += 1
       report["total_amount"] += order["amount"]

# Create a mutable summary report dictionary.
summary_report = {"count": 0, "total_amount": 0.0}

# Use a generator to stream 10,000 orders.
orders_gen = order_stream(10000)

# Use a list comprehension to filter orders that are 'completed' and have amount > 200.
high_value_completed_orders = [order for order in orders_gen
                              if order["status"] == "completed" and order["amount"] > 200]

# Update the summary report using our mutable dictionary.
update_summary(summary_report, high_value_completed_orders)

print("Summary Report for High-Value Completed Orders:")
print(summary_report)
```

如果我们想用 Java 达到相同的目标，由于 Java 缺乏内置的生成器和列表推导式，我们必须生成一个订单列表，然后使用显式循环进行过滤和更新总结，从而使代码更加复杂、可读性降低且难以维护。

```py
import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

class Order {
   public String orderId;
   public double amount;
   public String status;

   public Order(String orderId, double amount, String status) {
       this.orderId = orderId;
       this.amount = amount;
       this.status = status;
   }

   @Override
   public String toString() {
       return String.format("{orderId:%s, amount:%.2f, status:%s}", orderId, amount, status);
   }
}

public class OrderProcessor {
   // Generates a list of orders.
   public static List<Order> generateOrders(int numOrders) {
       List<Order> orders = new ArrayList<>();
       String[] statuses = {"completed", "pending", "cancelled"};
       Random rand = new Random();
       for (int i = 0; i < numOrders; i++) {
           String orderId = "ORD" + (i + 1);
           double amount = Math.round(ThreadLocalRandom.current().nextDouble(10.0, 500.0) * 100.0) / 100.0;
           String status = statuses[rand.nextInt(statuses.length)];
           orders.add(new Order(orderId, amount, status));
       }
       return orders;
   }

   // Filters orders based on criteria.
   public static List<Order> filterHighValueCompletedOrders(List<Order> orders) {
       List<Order> filtered = new ArrayList<>();
       for (Order order : orders) {
           if ("completed".equals(order.status) && order.amount > 200) {
               filtered.add(order);
           }
       }
       return filtered;
   }

   // Updates a mutable summary Map with the count and total amount.
   public static void updateSummary(Map<String, Object> summary, List<Order> orders) {
       int count = 0;
       double totalAmount = 0.0;
       for (Order order : orders) {
           count++;
           totalAmount += order.amount;
       }
       summary.put("count", count);
       summary.put("total_amount", totalAmount);
   }

   public static void main(String[] args) {
       // Generate orders.
       List<Order> orders = generateOrders(10000);

       // Filter orders.
       List<Order> highValueCompletedOrders = filterHighValueCompletedOrders(orders);

       // Create a mutable summary map.
       Map<String, Object> summaryReport = new HashMap<>();
       summaryReport.put("count", 0);
       summaryReport.put("total_amount", 0.0);

       // Update the summary report.
       updateSummary(summaryReport, highValueCompletedOrders);

       System.out.println("Summary Report for High-Value Completed Orders:");
       System.out.println(summaryReport);
   }
}
```

## 结论

配备了动态类型、列表推导式、生成器以及其参数传递和可变性的处理方式，Python 正在使其编码更加简化，同时增强内存效率和性能。因此，Python 已经成为自学者的理想编程语言。

感谢阅读！
