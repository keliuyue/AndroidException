### 让人欲罢不能的空指针（NullPointerException）

#### 定义：NullPointerException是java.lang.NullPointerException的简称，是Java语言中的一个异常类，位于java.lang包中，父类是java.lang.RuntimeException，该异常在源程序中可以不进行捕获和处理。

#### 发生频率： ★★★★★

#### 发生原因：当应用程序试图在需要对象的地方使用时，抛出该异常。这种情况包括：

- 调用 null 对象的实例方法。
- 访问或修改 null 对象的字段。
- 将 null 作为一个数组，获得其长度。
- 将 null 作为一个数组，访问或修改其时间片。
- 将 null 作为 Throwable 值抛出。
- Android 中View没有初始化

应用程序应该抛出该类的实例，指示其他对 null 对象的非法使用。

#### 1.简单例子分析

##### 1.1 具体事例

现在看一个我们经常在log中出现的事例，下面我们看一下这个Log

````
 Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
````

##### 1.2 分析

我们简单分析一下，这个是我们所遇到的问题。最上面一行表示我们遇到的是`java.lang.NullPointerException`问题，然后我们需要特别关注`at`这个单词，它可以告诉我们问题发生的具体位置。然后我们找到相应的位置并解决问题。我们最上面的`at` 提示的是

```
 at com.example.myproject.Book.getTitle(Book.java:16)
```

我们通过debug，首先找到`Book.java`类，然后看一下第16行

```
15   public String getTitle() {
16      System.out.println(title.toString());
17      return title;
18   }
```

通过上面的代码，因为我们报错的方法是`getTitle`,然后这个方法返回的`String`值，如果为空的化，只有可能`title` 为空。所以可以了解到基本上就是`title` 为`null`

#### 2.一连串exceptions的例子

##### 2.1具体事例

有时候APP catch的是一个Exception，然而实际抛出的是另一个Exception，像这样的：

```
34   public void getBookIds(int id) {
35      try {
36         book.getId(id);    // 这个方法抛出NullPointerException在22行
37      } catch (NullPointerException e) {
38         throw new IllegalStateException("A book has a null property", e)
39      }
40   }
```

然后这个异常的Log是这样的：

```
Exception in thread "main" java.lang.IllegalStateException: A book has a null property
        at com.example.myproject.Author.getBookIds(Author.java:38)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
Caused by: java.lang.NullPointerException
        at com.example.myproject.Book.getId(Book.java:22)
        at com.example.myproject.Author.getBookIds(Author.java:36)
        ... 1 more
```

##### 2.2分析

这个异常中的`Caused by`有什么不同吗，有时候Log中有很多`Caused by`。我们希望在这些错误中找到最主要的问题，一般主要的问题出现在最后的`Caused by`。在这个异常中的主要问题是

```
Caused by: java.lang.NullPointerException <-- 主要导致的Exception
        at com.example.myproject.Book.getId(Book.java:22) <-- 最重要的一行
```

然后我们根据这个异常Log，我们可以定位到`Book.java`类中22行的`getId`方法。确定是这行导致的`NullPointerException` 。然后判断是否有值为`null` 。

#### 3.方法中对象参数为空导致的NullPointerException
##### 3.1 具体事例

```
public void doSomething(SomeObject obj){
   //一些对obj的操作
}
```

##### 3.2分析

正常看上去没有什么问题，但是这个会出现隐藏的bug，平时我们在编码过程中很容易出现这种细节的问题。当我们使用下列代码的时候。

```
doSomething(null);
```

当你传递的参数对象`SomeObject`为null的时候，例如：

```
SomeObject obj = null;
doSomething(obj);
// 等同于
doSomething(null);
```

最后实现的就是这样的。当方法中使用`SomeObject`对象时。会发生`NullPointerException`异常。所以在我们操作的过程中。我们需要对传递过来的`SomeObject`对象进行非空判断，然后在对象不为空的时候进行你需要的操作，然后在为空的时候进行提示或者其他操作。

所以最后我们的处理操作是：

```
/**
  * @param obj 参数有可能为空
  *
  */
public void doSomething(SomeObject obj){
    if(obj != null){
       //do something
    } else {
       //do something else
    }
}
```

#### 4.创建对象数组时候抛出空指针

##### 4.1具体事例

```
public class ResultList {
    public String name;
    public Object value;

    public ResultList() {}
}
```

```
public class Test {
    public static void main(String[] args){
        ResultList[] boll = new ResultList[5];
        boll[0].name = "iiii";
    }
}
```
##### 4.2分析
首先看一下`main`方法中的这一行代码

```
ResultList[] boll = new ResultList[5];
```
对象数组已经初始化了，但是数据中的每一个对象并没有初始化。所以我们调用`boll[0].name = "iiii";`这行代码会报空指针。
我们应该在调用之前进行初始化对象操作。

```
ResultList[] boll = new ResultList[5];
// 调用之前进行初始化
boll[0] = new ResultList(); 
boll[0].name = "iiii";
```

或者另外一种方式初始化

```
ResultList[] boll = {new ResultList(),new ResultList(),new ResultList(),new ResultList(),new ResultList()}；
// 这种数据的定义并对每一个对象进行初始化
boll[0].name = "iiii";
```

这种方式就是数据的定义和初始化的两种方式。

#### 5.在Fragment中调用onCreate方法并找控件造成NullPointerException

##### 5.1具体事例

`Fragment`的`onCreate()`方法

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    View something = findViewById(R.id.something);
    something.setOnClickListener(new View.OnClickListener() { ... }); // NPE HERE

    if (savedInstanceState == null) {
        getSupportFragmentManager().beginTransaction()
                .add(R.id.container, new PlaceholderFragment()).commit();
    }
}
```

`Fragment`的布局

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="packagename.MainActivity$PlaceholderFragment" >

    <View
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:id="@+id/something" />

</RelativeLayout>
```

##### 5.2分析

首先我们看一张Fragment生命周期的图片

![](https://i.stack.imgur.com/Z3B7j.png)

可以知道`onCreate`方法在`onCreateView`方法之前执行。然后返回null。然后如果调用方法就造成了`NullPointerException`
可以使用以下的方法进行

```
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
    Bundle savedInstanceState) {
  View rootView = inflater.inflate(R.layout.fragment_main, container,
      false);

  View something = rootView.findViewById(R.id.something); // not activity findViewById()
  something.setOnClickListener(new View.OnClickListener() { ... });

  return rootView;
}
```

我们等到布局已经创建之后进行`findViewById`就不会造成`NullPointerException`

#### 6.使用RecyclerView造成的NullPointerException

##### 6.1具体事例

报错日志信息

```
java.lang.NullPointerException: Attempt to invoke virtual method 'void android.support.v7.widget.RecyclerView$LayoutManager.onMeasure(android.support.v7.widget.RecyclerView$Recycler, android.support.v7.widget.RecyclerView$State, int, int)' on a null object reference
        at android.support.v7.widget.RecyclerView.onMeasure(RecyclerView.java:1764)
        at android.view.View.measure(View.java:17430)
```

##### 6.2分析
根据我们上面使用的方法，找到报错行文件。找到有可能发生`NullPointerException`的位置

```
void android.support.v7.widget.RecyclerView$LayoutManager.onMeasure
```

是没有提供`LayoutManager`造成的，所以我们需要做以下处理

```
// 还需要找到相应的recyclerView
final LinearLayoutManager layoutManager = new LinearLayoutManager(context);
layoutManager.setOrientation(LinearLayoutManager.VERTICAL);
recyclerView.setLayoutManager(layoutManager);
```

#### 7.findViewById方法引发的NullPointerException

##### 7.1具体事例

`MainActivity`

```
public class MainActivity extends Activity {

    ViewPager pager;
    MyPagerAdapter adapter;
    LinearLayout layout1, layout2, layout3;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        layout1 = (LinearLayout) findViewById(R.id.first_View);
        layout2 = (LinearLayout) findViewById(R.id.second_View);
        layout3 = (LinearLayout) findViewById(R.id.third_View);

        adapter = new MyPagerAdapter();
        pager = (ViewPager) findViewById(R.id.main_pager);
        pager.setAdapter(adapter);
    }

    private class MyPagerAdapter extends PagerAdapter
    {

        @Override
        public int getCount() { 
            return 3;
        }

        @Override
        public Object instantiateItem(ViewGroup collection, int position) {

            LinearLayout l = null;

            if (position == 0 )
            {
                l = layout1;
            }
            if (position == 1)
            {
                l = layout2;
            }

            if (position == 2)
            {
                l = layout3;
            }
                collection.addView(l, position);
                return l;
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return (view==object);
        }

         @Override
         public void destroyItem(ViewGroup collection, int position, Object view) {
                 collection.removeView((View) view);
         }
    }
}
```
      
`activity_main layout`

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:orientation="vertical"
                android:layout_width="fill_parent"
                android:layout_height="fill_parent"
                android:background="#a4c639">


    <android.support.v4.view.ViewPager
                        android:layout_width="match_parent" 
                        android:layout_height="match_parent" 
                        android:id="@+id/main_pager"/>
</LinearLayout>
```

`activity_first layout`

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/first_View">

<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />

<Button
    android:id="@+id/button1"
    style="?android:attr/buttonStyleSmall"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button" />

</LinearLayout>
```

`activity_second layout`

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/second_View">

<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />

</LinearLayout>
```

`And the activity_third layout`

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/third_View">

<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />

</LinearLayout>
```

##### 7.2分析

首先`findViewById`在`setContentView`方法调用完之后调用，并且在`setContentView`中找需要的控件，如果没有则返回为null。

**Sample**

当使用`setContentView(R.layout.activity_first);`在你的代码中

如果你调用`layout1 = (LinearLayout) findViewById(R.id.first_View);`会返回一个View（因为你的`activity_first`存在这个控件，并有这个`id`）

如果你调用`layout2 = (LinearLayout) findViewById(R.id.second_View);`会返回null（因为在`activity_first`中找不到这个控件的id）

所以在我们使用控件的时候，如果是`findViewById`造成的`NullPointerException`,你就需要在布局文件中寻找是否有该控件和id。

#### 总结

`NullPointerException`发生的原因有很多种，我们也有相应的解决方法。主要的方式有：

- 首先查看报`NullPointerException`的日志，然后找到具体的位置(1.找到at的位置，2.定位你个人的包名，如2.1中`at com.example.myproject.Author.getBookIds`)，然后进行非空判断或者其他操作

- 如果日志中没有找到具体的位置，然后需要找到具体报错的外部类名和方法（如6.1中`android.support.v7.widget.RecyclerView$LayoutManager.onMeasure`），然后进行非空操作或者其他操作

- Android `Activity`和`Fragment`中出现`NullPointerException`,如果以上方法没有问题，了解他们的生命周期，然后看看执行顺序有没有问题。

- 数组，对象，集合等调用方法之前需要初始化，`List<String> mList = new ArrayList<>();`这种类似的操作必不可少。

##### github原地址：https://github.com/keliuyue/AndroidException/blob/master/contents/java/NullPointerException.md

### 参考
1. Android研发录

1. [https://stackoverflow.com/questions/218384/what-is-a-nullpointerexception-and-how-do-i-fix-it/218510218510](https://stackoverflow.com/questions/218384/what-is-a-nullpointerexception-and-how-do-i-fix-it/218510#218510)

2. [https://stackoverflow.com/questions/1922677/nullpointerexception-when-creating-an-array-of-objects](https://stackoverflow.com/questions/1922677/nullpointerexception-when-creating-an-array-of-objects)

3. [https://stackoverflow.com/questions/23653778/nullpointerexception-accessing-views-in-oncreate](https://stackoverflow.com/questions/23653778/nullpointerexception-accessing-views-in-oncreate)

4. [https://stackoverflow.com/questions/3988788/what-is-a-stack-trace-and-how-can-i-use-it-to-debug-my-application-errors](https://stackoverflow.com/questions/3988788/what-is-a-stack-trace-and-how-can-i-use-it-to-debug-my-application-errors)

6. [https://stackoverflow.com/questions/19078461/null-pointer-exception-findviewbyid](https://stackoverflow.com/questions/19078461/null-pointer-exception-findviewbyid)