**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**

# 前言： 
2015年谷歌I/O大会上介绍了一个数据绑定框架DataBinding。2016年，2017年毫无意外成了项目实战中主流框架。使用它我们可以轻松实现MVVM（模型-视图-视图模型）模式，来实现应用之间数据与视图的分离、视图与业务逻辑的分离、数据与业务逻辑的分离，从而达到低耦合、可重用性、易测试性等好处。而使用DataBinding不仅减少了findViewById的出现频率，而且还大大提高解析XML的速度。 

## 1 . 基本使用 
新建一个项目，在app的build文件加上：
```
    android {
    ...
    dataBinding{
        enabled =true;
    }
    ...
}
```
构建环境特别简单，接下来直接开始使用数据绑定 
首先，新建一个User.java实体类：
```
public class User {
    public String name;
    public String myBlog;
    public int age;

    public User(String name,int age,String myBlog){
        this.name=name;
        this.age=age;
        this.myBlog=myBlog;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getMyBlog() {
        return myBlog;
    }

    public void setMyBlog(String myBlog) {
        this.myBlog = myBlog;
    }
}
```
之后看下布局(activity_basic.xml)，跟传统的布局不一样，这里需要使用<layout></layout>作为根节点，在<layout>节点中我们可以通过<data>节点来引入我们要使用的数据源
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"
            type="com.donkor.demo.databinding.bean.User" />

    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">


        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(user.age)}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:autoLink="all"
            android:text="@{user.myBlog}" />

    </LinearLayout>

</layout>
```
**注意 :** 
※ 切记，在<layout>节点下是没有“layout_width”和“layout_height”的

※ type中声明的就是我们的用户实体类User，连同包名一定要写全！！！我们给其命名（name）为“user”,然后在TextView中的@{user.name}就是把这个user中的名字展示出来，之后的“age””myBlog”同理

※ age年龄在实体类User中我们定义的是整数类型，所以在布局中需要使用String.valueOf(user.age)转换为字符串

一个Binding类会基于layout文件的名称而产生，并且添加“Binding”后缀。上述的layout文件是main_basic.xml，因此生成的类名是ActivityBasicBinding。通过DataBindingUtil.setContentView获取bing实例，进行数据绑定
```
public class BasicActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityBasicBinding bing= DataBindingUtil.setContentView(this, R.layout.activity_basic);
        User user=new User("donkor",10,"http://blog.csdn.net/donkor_");
        bing.setUser(user);
    }
}
```
运行之后看下效果图 
![](https://upload-images.jianshu.io/upload_images/19956127-baec7f3c3a4b0d06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 2 . 显示照片
![](https://upload-images.jianshu.io/upload_images/19956127-0dfc86ed53c84ca1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看完图之后，我们看下如何实现。 
我们选择用glide加载一张网络图片。glide怎么在studio中使用，这里就不再讨论。如何显示一张网络图片，需要使用到DataBinding自定义属性@BindingAdapter。在布局文件中使用这个自定义属性，显示出我们要展示的布局。 
还是那个实体类User，添加自定义属性@BindingAdapter({"imageUrl"})
```
public class User {
    public String imageUrl;

    public User(String imageUrl){
        this.imageUrl=imageUrl;
    }

    /**
     * 使用ImageLoader显示图片
     * @param imageView
     * @param url
     */
    @BindingAdapter({"imageUrl"})
    public static void imageLoader(ImageView imageView, String url) {
        Glide.with(imageView.getContext()).load(url).into(imageView);
    }
}
```
※ 这里要主要的是，imageLoader方法必须声明是static，否则会报错。

新建activity_image.xml，因为使用了自定义属性，所以在layout节点下要添加xmlns:app="http://schemas.android.com/apk/res-auto"。在ImageView中使用app:imageUrl="@{user.imageUrl}"。完整代码如下：
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="user"
            type="com.donkor.demo.databinding.bean.User" />

    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:imageUrl="@{user.imageUrl}" />

    </LinearLayout>

</layout>
```
最后运行之后，结果如上图。我就不再发一遍了，有兴趣的朋友再拖回去看一遍，反正下面还有~~

## 3 . 更多用法 
**简单的字符拼接**
```
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{`my Name is :`+user.name}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:autoLink="all"
            android:text="@{`my Blog is :`+user.myBlog}" />
```
**简单的三目运算** 
判断名字是否为空，不为空只显示user.name，否则显示donkor11：
```
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text='@{user.name ?? "donkor11"}' />
```
相当于
```
<TextView
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:text='@{user.name !=null?user.name: "donkor11"}' />
```
 这里需要注意的是当{}中使用了双引号“”，最外层要改成单引号” 
**根据数据判断，显示数据** 
判断是否为学生，是则显示11，反则，显示00
```
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.isStudent?String.valueOf(11):String.valueOf(00)}" />
```
**修改样式 **
判断是否为学生，是则修改背景颜色0xFF0000FF，反则，显示0xFFFF0000
```
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="@{user.isStudent?0xFF0000FF:0xFFFF0000}"
    android:text="donkor" />
```
 写在之后的话，这里我们需要知道Databinding支持与不支持的表达式，语法。如下 
**支持的表达式：**
1.Mathematical + - / * %
2.String concatenation +
3.Logical && ||
4.Binary & | ^
5.Unary + - ! ~
6.Shift >> >>> <<
7.Comparison == > < >= <=
8.instanceof
9.Grouping ()
10.Literals - character, String, numeric, null
11.Cast
12.Method calls
13.Field access
14.Array access []
15.Ternary operator ?:

**不支持的表达式：**
1.this
2.super
3.new
4.Explicit generic invocation
最后运行之后，看下效果图。 
![](https://upload-images.jianshu.io/upload_images/19956127-9385b9fb3873a259.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
## 4 . 点击事件 
**单击事件** 
单击事件在实际开发中，使用频率有多高这里就不解释了。下面我们直接看怎么用 
在data节点下的variable下type引用android.view.View.OnClickListener。
```
<data>
    <variable
        name="myClick"
        type="android.view.View.OnClickListener" />
</data>
```
Button按钮直接引用”myClick”
```
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="单击按钮"
    android:onClick="@{myClick}" />
```
**长按事件** 
长按事件的button，首先需要分配一个id给Button
```
<Button
    android:id="@+id/btnLongClick"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="长按按钮"
    />
```
在Activity中，使用数据绑定，直接引用按钮id。完整代码如下
```
public class ClickActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityClickBinding bing=DataBindingUtil.setContentView(this, R.layout.activity_click);

        //事件绑定 -- 单击
        bing.setMyClick(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(ClickActivity.this,"发生了点击事件",Toast.LENGTH_SHORT).show();
            }
        });

        //事件绑定 -- 长按
        bing.btnLongClick.setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View v) {
                Toast.makeText(ClickActivity.this,"发生了长按事件",Toast.LENGTH_SHORT).show();
                return false;
            }
        });
    }
}
```
最后运行之后，看下效果图。 
![](https://upload-images.jianshu.io/upload_images/19956127-9c294da50227bee5.gif?imageMogr2/auto-orient/strip)


▲5 . 绑定ListView 
Listview在实际开发中使用频率同样很高，看了上面那么多例子。就知道Databinding绑定ListView一样简单。接下来看一组美图。
 ![](https://upload-images.jianshu.io/upload_images/19956127-b5aaaab041934b0d.gif?imageMogr2/auto-orient/strip)

看完之后，能看出布局其实就是一张ImageView，底下加一个TextView。例子很简单 ，然后看下如何实现。首先需要新建一个美女的实体类，我们先定义Beauty,beautyNum就是美女底下的TextView文本。
```
public class Beauty {
    public String beautyNum;
    public String imageUrl;

    public Beauty(String beautyNum, String imageUrl) {
        this.beautyNum = beautyNum;
        this.imageUrl = imageUrl;
    }

    @BindingAdapter({"imageUrl"})
    public static void beautyImage(ImageView imageView, String url) {
        Glide.with(imageView.getContext()).load(url).into(imageView);
    }
}
```
然后看下主布局activity_listview.xml，在data节点下的variable下type引用android.widget.BaseAdapter。
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="adapter"
            type="android.widget.BaseAdapter" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ListView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:adapter="@{adapter}" />
    </LinearLayout>
</layout>
```
然后ListView的item布局item_listview.xml,同样比较简单，在这不做解释。
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="beauty"
            type="com.donkor.demo.databinding.bean.Beauty" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">


        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:imageUrl="@{beauty.imageUrl}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{beauty.beautyNum}" />

    </LinearLayout>

</layout>
```
有ListView的地方，十有八九就有适配器。这里介绍一个终极适配器的写法MyBaseAdapter。尽管这里仅仅是给美女使用的适配器，但是已经说明了是终极写法。我们就不叫它美女适配器了。
```
public class MyBaseAdapter<T> extends BaseAdapter {

    private List<T> list;
    private int layoutId;
    private int variableId;
    private LayoutInflater mInflater;

    public MyBaseAdapter(Context context, List<T> list, int layoutId, int variableId) {
        this.list = list;
        this.layoutId = layoutId;
        this.variableId = variableId;
        this.mInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return list.size() == 0 ? 0 : list.size();
    }

    @Override
    public Object getItem(int position) {
        return list.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewDataBinding dataBinding;
        if (convertView == null) {
            dataBinding = DataBindingUtil.inflate(mInflater, layoutId, parent, false);
        } else {
            dataBinding = DataBindingUtil.getBinding(convertView);
        }
        dataBinding.setVariable(variableId, list.get(position));

        return dataBinding.getRoot();
    }
}
```
然后解释下其中几个变量

Context context：上下文，不比多说
List list：传进来的数据集合，不解释
int layoutId： item布局的资源id
int variableId：系统自动生成的
※ 注意布局加载方式为DataBindingUtil类中的inflate方法 
最后看下ListViewActivity完整代码
```
public class ListViewActivity extends Activity {
    private List<Beauty> list;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityListviewBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_listview);

        list = new ArrayList<>();
        //加载数据
        initData();

        MyBaseAdapter<Beauty> adapter = new MyBaseAdapter<>(ListViewActivity.this, list, R.layout.item_listview, BR.beauty);
        binding.setAdapter(adapter);
    }

    void initData() {
        Beauty beauty1 = new Beauty("第一个美女", "http://img2.imgtn.bdimg.com/it/u=3988249408,1489015532&fm=21&gp=0.jpg");
        Beauty beauty2 = new Beauty("第二个美女", "http://img4.imgtn.bdimg.com/it/u=2579627311,3580753633&fm=21&gp=0.jpg");
        Beauty beauty3 = new Beauty("第三个美女", "http://img5.imgtn.bdimg.com/it/u=539171541,1245868076&fm=23&gp=0.jpg");
        Beauty beauty4 = new Beauty("第四个美女", "http://img1.imgtn.bdimg.com/it/u=3494499027,4116428522&fm=23&gp=0.jpg");
        Beauty beauty5 = new Beauty("第五个美女", "http://img4.imgtn.bdimg.com/it/u=645329305,336210525&fm=23&gp=0.jpg");
        list.add(beauty1);
        list.add(beauty2);
        list.add(beauty3);
        list.add(beauty4);
        list.add(beauty5);
    }
}
```
最后运行之后，结果如上图。有兴趣的朋友再拖回去看一遍，反正下面没有美女了~~ 
## 6 . 数据更新 
Databinding的数据更新同样惹人喜欢。最主要的是代码简洁明了，UI还同步更新。 
首先看下效果图 
 ![](https://upload-images.jianshu.io/upload_images/19956127-9840c850362d7615.gif?imageMogr2/auto-orient/strip)

数据更新可以使用如下两种方式，分别对学生文本和老师文本进行修改 
**让实体类(Student)继承自BaseObservable** 
给需要改变的字段的get方法添加上@Bindable注解，然后给需要改变的字段(例如name)的set方法加上notifyPropertyChanged(BR.name);字段number同理，在set方法内加上notifyPropertyChanged(BR.number);
```
public class Student extends BaseObservable {
    @Bindable
    public String name;
    @Bindable
    public String number;

    public Student() {
    }

    public Student(String name, String number) {
        this.name = name;
        this.number = number;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
        notifyPropertyChanged(BR.name);
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
        notifyPropertyChanged(BR.number);
    }
}
```
** 使用DataBinding提供的ObservableFields来创建实体类** 
Teacher实体类,这个更加简单，几行代码搞定
```
public class Teacher extends BaseObservable {
    public ObservableField<String> name = new ObservableField<>();
    public ObservableField<String> number = new ObservableField<>();

    public Teacher() {
    }

    public Teacher(String name, String number) {
        this.name.set(name);
        this.number.set(number);
    }
}
```
然后看下布局文件activity_data_notify代码,跟上文描述的基本类似，这里不再过多赘述。
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="student"
            type="com.donkor.demo.databinding.bean.Student" />
        <variable
            name="teacher"
            type="com.donkor.demo.databinding.bean.Teacher" />

    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:orientation="vertical">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{student.name}" />


        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{student.number}" />


        <Button
            android:id="@+id/btnStudent"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="30dp"
            android:layout_marginRight="30dp"
            android:layout_marginTop="10dp"
            android:text="点击按钮修改学生文本"
            />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{teacher.name}" />


        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{teacher.number}" />

        <Button
            android:id="@+id/btnTeacher"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="30dp"
            android:layout_marginRight="30dp"
            android:layout_marginTop="10dp"
            android:text="点击按钮修改老师文本"
            />
    </LinearLayout>

</layout>
```
最后看下DataNotifyActivity完整代码
```
public class DataNotifyActivity extends Activity {

    private Student student;
    private Teacher teacher;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityDataNotifyBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_data_notify);
        student = new Student("studen_donkor", "number_aa");
        binding.setStudent(student);
        teacher = new Teacher("teacher_donkor", "number_bb");
        binding.setTeacher(teacher);

        //点击按钮，更新数据,同时更新UI
        binding.btnStudent.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                student.setName("donkor is not student");
                student.setNumber("number_00000");
            }
        });

        binding.btnTeacher.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                teacher.name.set("donkor is not teacher");
                teacher.number.set("number_11111");
            }
        });
    }
}
```
## 7 . 数据集合 
DataBinding中给我们提供了一些现成的集合ObservableArrayList，ObservableArrayMap。这里不做介绍。

Demo_CSDN 下载地址 : http://download.csdn.net/detail/donkor_/9740166
原文链接：https://blog.csdn.net/donkor_/article/details/54598215
**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
