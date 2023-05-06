# Effect-Java-3
2. 当构造⽅法参数过多时使⽤ builder 模式
静态⼯⼚和构造⽅法都有⼀个限制︓它们不能很好地扩展到很多可选参数的情景。请考虑⼀个代表包装⾷品上
的营养成分标签的例⼦。这些标签有⼏个必需的属性——每次建议的摄⼊量，每罐的份量和每份卡路⾥ ，以及
超过 20 个可选的属性——总脂肪、饱和脂肪、反式脂肪、胆固醇、钠等等。⼤多数产品只有这些可选字段中
的少数，且具有⾮零值。
应该为这样的类编写什么样的构造⽅法或静态⼯⼚︖传统上，程序员使⽤了可伸缩（telescoping constructor）
构造⽅法模式，在这种模式中，⾸先提供⼀个只有必需参数的构造⽅法，接着提供增加了⼀个可选参数的构造
函数，然后提供增加了两个可选参数的构造函数，等等，最终在构造函数中包含所有必需和可选参数。以下就
是它在实践中的样⼦。为了简便起见，只显⽰了四个可选属性︓
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
 private final int servingSize; // (mL) required
 private final int servings; // (per container) required
 private final int calories; // (per serving) optional
 private final int fat; // (g/serving) optional
 private final int sodium; // (mg/serving) optional
 private final int carbohydrate; // (g/serving) optional
all.md 2023/4/30
4 / 248
 public NutritionFacts(int servingSize, int servings) {
 this(servingSize, servings, 0);
 }
 public NutritionFacts(int servingSize, int servings,
 int calories) {
 this(servingSize, servings, calories, 0);
 }
 public NutritionFacts(int servingSize, int servings,
 int calories, int fat) {
 this(servingSize, servings, calories, fat, 0);
 }
 public NutritionFacts(int servingSize, int servings,
 int calories, int fat, int sodium) {
 this(servingSize, servings, calories, fat, sodium, 0);
 }
 public NutritionFacts(int servingSize, int servings,
 int calories, int fat, int sodium, int carbohydrate) {
 this.servingSize = servingSize;
 this.servings = servings;
 this.calories = calories;
 this.fat = fat;
 this.sodium = sodium;
 this.carbohydrate = carbohydrate;
 }
}
当想要创建⼀个实例时，可以使⽤包含所有要设置的参数的最短参数列表的构造⽅法︓
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
通常情况下，这个构造⽅法的调⽤需要许多你不想设置的参数，但是你不得不为它们传递⼀个值。 在这种情况
下，我们为 fat 属性传递了 0 值。「只有」六个参数可能看起来并不那么糟糕，但随着参数数量的增加，它很
快就会失控。
简⽽⾔之，可伸缩构造⽅法模式是有效的，但是当有很多参数时，很难编写客户端代码，⽽且很难读懂它。 读
者不知道这些值是什么意思，并且必须仔细地去数参数才能找到答案。⼀长串相同类型的参数可能会导致⼀些
细微的 bug。如果客户端不⼩⼼写反了两个这样的参数，编译器并不会报错，但是程序在运⾏时会出现错误⾏
为 （详见第 51 条）。
当在构造⽅法中遇到许多可选参数时，另⼀种选择是 JavaBeans 模式，在这种模式中，调⽤⼀个⽆参的构造⽅
法来创建对象，然后调⽤ setter ⽅法来设置每个必需的参数和可选参数︓
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
 // Parameters initialized to default values (if any)
all.md 2023/4/30
5 / 248
 private int servingSize = -1; // Required; no default value
 private int servings = -1; // Required; no default value
 private int calories = 0;
 private int fat = 0;
 private int sodium = 0;
 private int carbohydrate = 0;
 public NutritionFacts() { }
 // Setters
 public void setServingSize(int val) { servingSize = val; }
 public void setServings(int val) { servings = val; }
 public void setCalories(int val) { calories = val; }
 public void setFat(int val) { fat = val; }
 public void setSodium(int val) { sodium = val; }
 public void setCarbohydrate(int val) { carbohydrate = val; }
}
这种模式没有伸缩构造⽅法模式的缺点。有点冗长，但创建实例很容易，并且易于阅读所⽣成的代码:
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
不幸的是，JavaBeans 模式本⾝有严重的缺陷。由于构造⽅法被分割成了多次调⽤，所以在构造过程中
JavaBean 可能处于不⼀致的状态。该类没有通过检查构造参数参数的有效性来强制⼀致性的选项。在不⼀致的
状态下尝试使⽤对象可能会导致⼀些错误，这些错误与平常代码的BUG很是不同，因此很难调试。⼀个相关的
缺点是，JavaBeans 模式排除了让类不可变的可能性（详见第 17 条），并且需要程序员增加⼯作以确保线程安
全。
通过在对象构建完成时⼿动「冻结」对象，并且不允许它在解冻之前使⽤，可以减少这些缺点，但是这种变体
在实践中很难使⽤并且很少使⽤。 ⽽且，在运⾏时会导致错误，因为编译器⽆法确保程序员会在使⽤对象之前
调⽤ freeze ⽅法。
幸运的是，还有第三种选择，它结合了可伸缩构造⽅法模式的安全性和 JavaBean 模式的可读性。 它是 Builder
模式[Gamma95] 的⼀种形式。客户端不直接构造所需的对象，⽽是调⽤⼀个包含所有必需参数的构造⽅法 (或
静态⼯⼚)得到获得⼀个 builder 对象。然后，客户端调⽤ builder 对象的与 setter 相似⽅法来设置你想设置的
可选参数。最后，客户端调⽤builder对象的⼀个⽆参的 build ⽅法来⽣成对象，该对象通常是不可变的。
Builder 通常是它所构建的类的⼀个静态成员类（详见第 24 条）。以下是它在实践中的⽰例︓
// Builder Pattern
public class NutritionFacts {
 private final int servingSize;
 private final int servings;
 private final int calories;
all.md 2023/4/30
6 / 248
 private final int fat;
 private final int sodium;
 private final int carbohydrate;
 public static class Builder {
 // Required parameters
 private final int servingSize;
 private final int servings;
 // Optional parameters - initialized to default values
 private int calories = 0;
 private int fat = 0;
 private int sodium = 0;
 private int carbohydrate = 0;
 public Builder(int servingSize, int servings) {
 this.servingSize = servingSize;
 this.servings = servings;
 }
 public Builder calories(int val) { 
 calories = val; 
 return this;
 }
 public Builder fat(int val) { 
 fat = val; 
 return this;
 }
 public Builder sodium(int val) { 
 sodium = val; 
 return this; 
 }
 public Builder carbohydrate(int val) { 
 carbohydrate = val; 
 return this; 
 }
 public NutritionFacts build() {
 return new NutritionFacts(this);
 }
 }
 private NutritionFacts(Builder builder) {
 servingSize = builder.servingSize;
 servings = builder.servings;
 calories = builder.calories;
 fat = builder.fat;
 sodium = builder.sodium;
 carbohydrate = builder.carbohydrate;
 }
}
all.md 2023/4/30
7 / 248
NutritionFacts 类是不可变的，所有的参数默认值都在⼀个地⽅。builder 的 setter ⽅法返回 builder 本⾝，
这样就可以进⾏链式调⽤，从⽽⽣成⼀个流畅的 API。下⾯是客户端代码的⽰例︓
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
 .calories(100).sodium(35).carbohydrate(27).build();
这个客户端代码很容易编写，更重要的是易于阅读。 采⽤Builder 模式模拟实现的的可选参数可以在Python和
Scala都可以找到。
为了简洁起见，省略了有效性检查。 要尽快检测⽆效参数，检查 builder 的构造⽅法和⽅法中的参数有效性。
在 build ⽅法调⽤的构造⽅法中检查包含多个参数的不变性。为了确保这些不变性不受攻击，在从 builder 复
制参数后对对象属性进⾏检查（详见第 50 条）。 如果检查失败，则抛出 IllegalArgumentException 异常
（详见第 72 条），其详细消息指⽰哪些参数⽆效（详见第 75 条）。
Builder 模式⾮常适合类层次结构。 使⽤平⾏层次的 builder，每个builder嵌套在相应的类中。 抽象类有抽象的
builder︔具体的类有具体的 builder。 例如，考虑代表各种⽐萨饼的根层次结构的抽象类︓
// Builder pattern for class hierarchies
import java.util.EnumSet;
import java.util.Objects;
import java.util.Set;
public abstract class Pizza {
 public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
 final Set<Topping> toppings;
 
 abstract static class Builder<T extends Builder<T>> {
 EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
 public T addTopping(Topping topping) {
 toppings.add(Objects.requireNonNull(topping));
 return self();
 }
 
 abstract Pizza build();
 
 // Subclasses must override this method to return "this"
 protected abstract T self();
 }
 Pizza(Builder<?> builder) {
 toppings = builder.toppings.clone(); // See Item 50
 }
}
all.md 2023/4/30
8 / 248
请注意，Pizza.Builder 是⼀个带有递归类型参数（ recursive type parameter）（详见第 30 条）的泛型类
型。 这与抽象的 self ⽅法⼀起，允许⽅法链在⼦类中正常⼯作，⽽不需要强制转换。 Java 缺乏⾃我类型的这
种变通解决⽅法被称为模拟⾃我类型（simulated self-type）。
这⾥有两个具体的 Pizza 的⼦类，其中⼀个代表标准的纽约风格的披萨，另⼀个是半圆形烤乳酪馅饼。前者有
⼀个所需的尺⼨参数，⽽后者则允许指定酱汁是否应该在⾥⾯或在外⾯︓
import java.util.Objects;
public class NyPizza extends Pizza {
 public enum Size { SMALL, MEDIUM, LARGE }
 private final Size size;
 public static class Builder extends Pizza.Builder<Builder> {
 private final Size size;
 public Builder(Size size) {
 this.size = Objects.requireNonNull(size);
 }
 @Override public NyPizza build() {
 return new NyPizza(this);
 }
 @Override protected Builder self() {
 return this;
 }
 }
 private NyPizza(Builder builder) {
 super(builder);
 size = builder.size;
 }
}
public class Calzone extends Pizza {
 private final boolean sauceInside;
 
 public static class Builder extends Pizza.Builder<Builder> {
 private boolean sauceInside = false; // Default
 public Builder sauceInside() {
 sauceInside = true;
 return this;
 }
 
 @Override public Calzone build() {
 return new Calzone(this);
 }
 
 @Override protected Builder self() {
 return this; 
all.md 2023/4/30
9 / 248
 }
 }
 
 private Calzone(Builder builder) {
 super(builder);
 sauceInside = builder.sauceInside;
 }
}
请注意，每个⼦类 builder 中的 build ⽅法被声明为返回正确的⼦类︓NyPizza.Builder 的 build ⽅法返回
NyPizza，⽽ Calzone.Builder 中的 build ⽅法返回 Calzone。 这种技术，其⼀个⼦类的⽅法被声明为返回
在超类中声明的返回类型的⼦类型，称为协变返回类型（covariant return typing）。 它允许客户端使⽤这些
builder，⽽不需要强制转换。
这些「分层 builder（hierarchical builders）」的客户端代码基本上与简单的 NutritionFacts builder 的代码
相同。为了简洁起见，下⾯显⽰的⽰例客户端代码假设枚举常量的静态导⼊︓
NyPizza pizza = new NyPizza.Builder(SMALL)
 .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
 .addTopping(HAM).sauceInside().build();
builder 对构造⽅法的⼀个微⼩的优势是，builder 可以有多个可变参数，因为每个参数都是在它⾃⼰的⽅法中
指定的。或者，builder 可以将传递给多个调⽤的参数聚合到单个属性中，如前⾯的 addTopping ⽅法所演⽰的
那样。
Builder 模式⾮常灵活。 单个 builder 可以重复使⽤来构建多个对象。 builder 的参数可以在构建⽅法的调⽤之
间进⾏调整，以改变创建的对象。 builder 可以在创建对象时⾃动填充⼀些属性，例如每次创建对象时增加的
序列号。
Builder 模式也有缺点。为了创建对象，⾸先必须创建它的 builder。虽然创建这个 builder 的成本在实践中不太
可能被注意到，但在看中性能的场合下这可能就是⼀个问题。⽽且，builder 模式⽐伸缩构造⽅法模式更冗长，
因此只有在有⾜够的参数时才值得使⽤它，⽐如四个或更多。但是请记住，你可能在以后会想要添加更多的参
数。但是，如果你⼀开始是使⽤的构造⽅法或静态⼯⼚，当类演化到参数数量失控的时候再转到Builder模式，
过时的构造⽅法或静态⼯⼚就会⾯临尴尬的处境。因此，通常最好从⼀开始就创建⼀个 builder。
总⽽⾔之，当设计类的构造⽅法或静态⼯⼚的参数超过⼏个时，Builder 模式是⼀个不错的选择，特别是如果许
多参数是可选的或相同类型的。builder模式客户端代码⽐使⽤伸缩构造⽅法（telescoping constructors）更容
易读写，并且builder模式⽐ JavaBeans 更安全。
