**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**

## 一、简介

Room是Google推出的Android架构组件库中的数据持久化组件库, 也可以说是在SQLite上实现的一套ORM解决方案。Room主要包含三个部分：

*   **Database** : 持有DB和DAO
*   **Entity** : 定义POJO类，即数据表结构
*   **DAO**(Data Access Objects) : 定义访问数据（增删改查）的接口

其关系如下图所示：

![](//upload-images.jianshu.io/upload_images/1712015-c8e826f33678b794.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

## 二、基本使用

### 1\. 创建Entity

#### 1.1 一个简单的Entitiy

一个简单Entity定义如下：

```
@Entity(tableName = "user" 
          indices = {@Index(value = {"first_name", "last_name"})})
public class User {

    @PrimaryKey
    private int uid;

    @ColumnInfo(name = "first_name")
    private String firstName;

    @ColumnInfo(name = "last_name")
    private String lastName;

    @Ignore
    public User(String firstName, String lastName) {
        this.uid = UUID.randomUUID().toString();
        this.firstName = firstName;
        this. lastName = lastName;
    }

    public User(String id, String firstName, String lastName) {
        this.uid = id;
        this.firstName = userName;
        this. lastName = userName;
    }

    // Getters and setters
}

```

*   `@Entity(tableName = "table_name**")` 注解POJO类，定义数据表名称;
*   `@PrimaryKey` 定义主键，如果一个Entity使用的是复合主键，可以通过`@Entity`注解的`primaryKeys` 属性定义复合主键：`@Entity(primaryKeys = {"firstName", "lastName"})`
*   `@ColumnInfo(name = “column_name”)` 定义数据表中的字段名
*   `@Ignore` 用于告诉Room需要忽略的字段或方法
*   建立索引：在`@Entity`注解的`indices`属性中添加索引字段。例如：`indices = {@Index(value = {"first_name", "last_name"}, unique = true), ...}`, `unique = true`可以确保表中不会出现`{"first_name", "last_name"}` 相同的数据。

#### 1.2 Entitiy间的关系

不同于目前存在的大多数ORM库，Room不支持Entitiy对象间的直接引用。（具体原因可以参考： [Understand why Room doesn't allow object references](https://link.jianshu.com?t=https://developer.android.com/training/data-storage/room/referencing-data.html#understand-no-object-references)）
但Room允许通过**外键(Foreign Key)**来表示Entity之间的关系。

```
@Entity(foreignKeys = @ForeignKey(entity = User.class,
                                  parentColumns = "id",
                                  childColumns = "user_id"))
class Book {
    @PrimaryKey
    public int bookId;

    public String title;

    @ColumnInfo(name = "user_id")
    public int userId;
}

```

如上面代码所示，Book对象与User对象是属于的关系。Book中的user_id,对应User中的id。 那么当一个User对象被删除时， 对应的Book会发生什么呢？

`@ForeignKey`注解中有两个属性`onDelete`和`onUpdate`， 这两个属性对应`ForeignKey`中的`onDelete()`和`onUpdate()`, 通过这两个属性的值来设置当User对象被删除／更新时，Book对象作出的响应。这两个属性的可选值如下：

*   `CASCADE`：User删除时对应Book一同删除； 更新时，关联的字段一同更新
*   `NO_ACTION`：User删除时不做任何响应
*   `RESTRICT`：禁止User的删除／更新。当User删除或更新时，Sqlite会立马报错。
*   `SET_NULL`：当User删除时， Book中的userId会设为NULL
*   `SET_DEFAULT`：与`SET_NULL`类似，当User删除时，Book中的userId会设为默认值

#### 1.3 对象嵌套

在某些情况下， 对于一张表中的数据我们会用多个POJO类来表示，在这种情况下可以用`@Embedded`注解嵌套的对象，比如：

```
class Address {
    public String street;
    public String state;
    public String city;

    @ColumnInfo(name = "post_code")
    public int postCode;
}

@Entity
class User {
    @PrimaryKey
    public int id;

    public String firstName;

    @Embedded
    public Address address;
}

```

以上代码所产生的User表中，Column 为`id, firstName, street, state, city, post_code`

### 2\. 创建数据访问对象（DAO）

```
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND "
           + "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);

    @Insert
    void insertAll(List<User> users);

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public void insertUsers(User... users);

    @Delete
    void delete(User user);

    @Update
    public void updateUsers(List<User> users);
}

```

DAO 可以是一个接口，也可以是一个抽象类， Room会在编译时创建DAO的实现。

**Tips:**

*   `@Insert`方法也可以定义返回值， 当传入参数仅有一个时返回`long`, 传入多个时返回`long[]`或`List<Long>`, Room在实现insert方法的实现时会在一个事务进行所有参数的插入。
*   `@Insert`的参数存在冲突时， 可以设置`onConflict`属性的值来定义冲突的解决策略， 比如代码中定义的是`@Insert(onConflict = OnConflictStrategy.REPLACE)`, 即发生冲突时替换原有数据
*   `@Update`和`@Delete` 可以定义`int`类型返回值，指更新／删除的函数

DAO中的增删改方法的定义都比较简单，这里不展开讨论，下面更多的聊一下查询方法。

#### 2.1 简单的查询

Talk is cheap, 直接show code:

```
@Query("SELECT * FROM user")
List<User> getAll();

```

> Room会在编译时校验sql语句，如果`@Query()` 中的sql语句存在语法错误，或者查询的表不存在，Room会在编译时报错。

#### 2.2 查询参数传递

```
@Query("SELECT * FROM user WHERE uid IN (:userIds)")
List<User> loadAllByIds(int[] userIds);

@Query("SELECT * FROM user WHERE first_name LIKE :first AND "
           + "last_name LIKE :last LIMIT 1")
User findByName(String first, String last);

```

看代码应该比较好理解， 方法中传递参数`arg`, 在sql语句中用`:arg`即可。编译时Room会匹配对应的参数。

> 如果在传参中没有匹配到`:arg`对应的参数, Room会在编译时报错。

#### 2.3 查询表中部分字段的信息

在实际某个业务场景中， 我们可能仅关心一个表部分字段的值，这时我仅需要查询关心的列即可。

定义子集的POJO类:

```
public class NameTuple {
    @ColumnInfo(name="first_name")
    public String firstName;

    @ColumnInfo(name="last_name")
    public String lastName;
}

```

在DAO中添加查询方法：

```
@Query("SELECT first_name, last_name FROM user")
public List<NameTuple> loadFullName();

```

> 这里定义的POJO也支持使用`@Embedded`

#### 2.3 查询结果的返回类型

Room中查询操作除了返回POJO对象及其List以外， 还支持：

*   **`LiveData<T>`**:
    LiveData是架构组件库中提供的另一个组件，可以很好满足数据变化驱动UI刷新的需求。Room会实现更新LiveData的代码。

```
@Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)") 
public LiveData<List<User>> loadUsersFromRegionsSync(List<String> regions);

```

*   **`Flowablbe<T>`** **`Maybe<T>`** **`Single<T>`**:
    Room 支持返回RxJava2 的`Flowablbe`, `Maybe`和`Single`对象，对于使用RxJava的项目可以很好的衔接， 但需要在gradle添加该依赖：`android.arch.persistence.room:rxjava2`。

```
@Query("SELECT * from user where id = :id LIMIT 1")
public Flowable<User> loadUserById(int id);

```

*   **`Cursor`**:
    返回Cursor是为了支持现有项目中使用Cursor的场景，官方不建议直接返回Cursor.

> Caution: It's highly discouraged to work with the Cursor API because it doesn't guarantee whether the rows exist or what values the rows contain. Use this functionality only if you already have code that expects a cursor and that you can't refactor easily.

#### 2.4 联表查询

Room支持联表查询，接口定义上与其他查询差别不大， 主要还是sql语句的差别。

```
@Dao
public interface MyDao {
    @Query("SELECT * FROM book "
           + "INNER JOIN loan ON loan.book_id = book.id "
           + "INNER JOIN user ON user.id = loan.user_id "
           + "WHERE user.name LIKE :userName")
   public List<Book> findBooksBorrowedByNameSync(String userName);
}

```

### 3\. 创建数据库

Room中DataBase类似SQLite API中SQLiteOpenHelper，是提供DB操作的切入点，但是除了持有DB外， 它还负责持有相关数据表（Entity）的数据访问对象（DAO）, 所以Room中定义Database需要满足三个条件：

*   继承RoomDataBase，并且是一个抽象类
*   用@Database 注解，并定义相关的entity对象， 当然还有必不可少的数据库版本信息
*   定义返回DAO对象的抽象方法

```
@Database(entities = {User.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}

```

创建好以上Room的三大组件后， 在代码中就可以通过以下代码创建Database实例。

```
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name").build();

```

## 三、数据库迁移

### 3.1 Room数据库升级

在传统的SQLite API中，我们如果要升级数据库， 通常在`SQLiteOpenHelper.onUpgrade`方法执行数据库升级的sql语句，这些sql语句的通常根据数据库版本以文件的方式或者用数组来管理。有人说这种方式升级数据库就像在拆炸弹，相比之下在Room中升级数据库简单的就像是按一个开关而已。

Room提供了Migration类来实现数据库的升级:

```
Room.databaseBuilder(getApplicationContext(), MyDb.class, "database-name")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3).build();

static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE `Fruit` (`id` INTEGER, "
                + "`name` TEXT, PRIMARY KEY(`id`))");
    }
};

static final Migration MIGRATION_2_3 = new Migration(2, 3) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE Book "
                + " ADD COLUMN pub_year INTEGER");
    }
};

```

在创建Migration类时需要指定`startVersion`和`endVersion`, 代码中`MIGRATION_1_2`和`MIGRATION_2_3`的startVersion和endVersion是递增的， Migration其实是支持从版本1直接升到版本3，只要其`migrate()`方法里执行的语句正常即可。那么Room是怎么实现数据库升级的呢？其实本质上还是调用`SQLiteOpenHelper.onUpgrade`，Room中自己实现了一个`SQLiteOpenHelper`， 在`onUpgrade()`方法被调用时触发`Migration`，当第一次访问数据库时，Room做了以下几件事：

*   创建Room Database实例
*   `SQLiteOpenHelper.onUpgrade`被调用，并且触发`Migration`
*   打开数据库

这样一看， Room中处理数据库升级确实很像是加一个开关。

### 3.2 原有SQLite数据库迁移至Room

因为Room使用的也是SQLite， 所以可以很好的支持原有Sqlite数据库迁移到Room。

假设原有一个版本号为1的数据库有一张表User, 现在要迁移到Room， 我们需要定义好Entity, DAO, Database, 然后创建Database时添加一个空实现的Migraton即可。需要注意的是，即使对数据库没有任何升级操作，也需要升级版本， 否则会抛异常`IllegalStateException`.

```
@Database(entities = {User.class}, version = 2)
public abstract class UsersDatabase extends RoomDatabase {
…
static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        // Since we didn't alter the table, there's nothing else to do here.
    }
};
…
database =  Room.databaseBuilder(context.getApplicationContext(),
        UsersDatabase.class, "Sample.db")
        .addMigrations(MIGRATION_1_2)
        .build();

```

## 四、复杂数据的处理

在某些场景下我们的应用可能需要存储复杂的数据类型，比如`Date`，但是Room的Entity仅支持基本数据类型和其装箱类之间的转换，不支持其它的对象引用。所以Room提供了`TypeConverter`给使用者自己实现对应的转换。

一个`Date`类型的转换如下：

```
public class Converters {
    @TypeConverter
    public static Date fromTimestamp(Long value) {
        return value == null ? null : new Date(value);
    }

    @TypeConverter
    public static Long dateToTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
}

```

定义好转换方法后，指定到对应的Database上即可， 这样就可以在对应的POJO（User）中使用`Date`类了。

```
@Database(entities = {User.class}, version = 1)
@TypeConverters({Converters.class})
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}

```

```
@Entity
public class User {
    ...
    private Date birthday;
}

```

## 五、总结

在SQLite API方式实现数据持久化的项目中，相信都有一个任务繁重的`SQLiteOpenHelper`实现, 一堆维护表的字段的`Constant`类， 一堆代码类似的数据库访问类（DAO），访问数据库时需要做Cursor的遍历，构建并返回对应的POJO类...相比之下，Room作为在SQLite之上封装的ORM库确实有诸多优势，比较直观的体验是：

*   比SQLite API更简单的使用方式
*   省略了许多重复代码
*   能在编译时校验sql语句的正确性
*   数据库相关的代码分为Entity, DAO, Database三个部分，结构清晰
*   简单安全的数据库升级方案

想要了解更多Room相关内容可以戳下面的链接：

*   Google Sample [https://github.com/googlesamples/android-architecture-components](https://link.jianshu.com?t=https://github.com/googlesamples/android-architecture-components)
*   Room数据库迁移[https://medium.com/google-developers/understanding-migrations-with-room-f01e04b07929](https://link.jianshu.com?t=https://medium.com/google-developers/understanding-migrations-with-room-f01e04b07929)
*   Room使用引导说明 [https://medium.com/google-developers/7-steps-to-room-27a5fe5f99b2](https://link.jianshu.com?t=https://medium.com/google-developers/7-steps-to-room-27a5fe5f99b2)
*   Room 🔗 RxJava [https://medium.com/google-developers/room-rxjava-acb0cd4f3757](https://link.jianshu.com?t=https://medium.com/google-developers/room-rxjava-acb0cd4f3757)
*   7 Pro-tips for Room [https://medium.com/google-developers/7-pro-tips-for-room-fbadea4bfbd1](https://link.jianshu.com?t=https://medium.com/google-developers/7-pro-tips-for-room-fbadea4bfbd1)

原文链接：https://www.jianshu.com/p/654d883e6ed0
**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**

