使用Builder应对多参数Constructor
======
static Factory 和 Constructor的共同问题：面对大量[Optional]参数 棘手.

: [Telescoping constructor]:第1个构造器仅含有必要参数，第N+1个构造器含有N个可选参数.
----
``` java
// Telescoping constructor pattern
public class NutritionFacts{
    private final int servingSize;  // required
    private final int servings;     // required
    private final int calories;     // optional 
    private final int fat;          // optional
    private final int sodium;       // optional 
    private final int carbohydrate; // optional

    public NutritionFacts(int servingSize, int servings){
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories){
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat){
        this(servingSize, servings, calories, fat, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium){
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate){
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
缺点： 复杂度随着参数数量增加爆炸； 可读性差； 一串连续的相同类型的参数会导致编译器无法发现的错误。

:[JavaBeans pattern]
----
确保Attribute保持非final + 非可选attribute通过非法值初始化.首先通过一个无参构造器创建Object,再使用setter。
``` java
public class NutritionFacts {
    private int servingSize = -1; // Required; no default value
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    //无参数构造器
    public NutritionFacts(){
    }

    //Setters
    public void setServingSize(int servingSize){
        this.servingSize = servingSize;
    }
    public void setServings(int servings){
        this.servings = servings;
    }
    public void setCalories(int calories){
        this.calories = calories;
    }
    public void setFat(int fat){
        this.fat = fat;
    }
    public void setSodium(int sodium){
        this.sodium = sodium;
    }
    public void setCarbohydrate(int carbohydrate){
        this.carbohydrate = carbohydrate;
    }
}

//调用 be like
NutritionFacts cola = new NutritionFacts();
cola.setServingSize(240);
cola.setServings(8);
...
...
```
缺点:
javaBeans杜绝了把Class做成Final的可能性 -> 无法直接保证线程安全

//?TODO: 如何理解在构造过程中JavaBeans可能处于不一致的状态。

:[Builder pattern]
-----
不直接生成需要的Object, 而是通过输入所有必要参数先创造Builder, 再在Builder上利用类Setter设置可选参数（流式）。
最后Builder返回（通常是）不可变的Object.
``` java
public class NutritionFacts{
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    //serve as method later only for[build()] 
    //把Builder收集到的可选参数聚合到Nutrition object
    public NutritionFacts(Builder builder){
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder{
        //Required parameters
        private final int servingSize;
        private final int servings;

        //Optional parameters - initialized to default values
        private int calories        = 0;
        private int fat             = 0;
        private int sodium          = 0;
        private int carbohydrate    = 0;

        //通过必要参数构建Builder
        public Builder(int servingSize, int servings){
            this.servingSize = servingSize;
            this.servings = servings;
        }
         public Builder calories(int val){
            calories = val;
            return this;
        }

        public Builder fat(int val){
            fat = val;
            return this;
        }

        public Builder sodium(int val){
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val){
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build(){
            return new NutritionFacts(this);
        }
    }
}

// Call be like:
NutritionFacts cola = new NutritionFacts.Builder(240,0) //required
        .calories(100)
        .carbohydrate(27)
        .....
        .build(); //终结

```
