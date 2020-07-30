Google最近出了个数据库框架名字叫Room Persistence Library。初步学习了一下，看起来还不错。

以前一直用原生的sqlite+contentprovider，原生的东西，靠近底层，api控制够细, 但是在需要自己写解析cursor。也用过litepal，这个是对sqlite轻量级的封装，不用再写解析代码，也不用写升级代码，结构简单，但是基于运行时注解效率较低。也用了一下realm,这个这两年非常火的数据库工具库，不再基于sqlite,也不是关系型数据库，有两个个巨大的毛病，查询出来的object不能再修改，而且增删改查必须在数据库对象创建的线程中才能使用，这个科非常蛋疼，因为通常为保证单例，会在application中实例化，这就导致后边的增删改查必须都在主线程中进行。在看[ Architecture Components](https://developer.android.com/topic/libraries/architecture/room.html " Architecture Components") 相关文章中了解到room这个框架，实验了一下，没有上述缺点，赶紧入门一下。

# 引入dependencies
首先项目的根build.gradle中添加
```groovy
allprojects {
    repositories {
        jcenter()
        google() // 添加google 的maven库
    }
}
```
在modules的build.gradle中添加
```groovy
dependencies {
    compile 'android.arch.persistence.room:runtime:1.0.0-alpha5' //api库，目前最新为alpha5

annotationProcessor "android.arch.persistence.room:compiler:1.0.0-alpha5" //注解处理库
	
	
    compile 'android.arch.persistence.room:rxjava2:1.0.0-alpha5' // 对于rxjava的支持库
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.1'
}
```
# 结构简介
room主要api由三个注解组成
database,dao,entity
database描述数据库版本，有哪些表和怎么获取dao。
dao是用来怎样从数据库增删改查，
entity这个一看名字就懂了，描述实体，对应数据库的表
三者的关系如下图

![关系](http://www.goinbowl.com/wp-content/uploads/2017/07/room_architecture.png "关系")

## entity
```java
@Entity(tableName = "users", primaryKeys = "name",indices={@Index("name")})
public class User {

    public int id;
    public String name;
    @ColumnInfo(name = "address")
    public String addr;
    public String phone;

    @Ignore
    public String nick;

    public User(int id, String name, String addr, String phone) {
        this.id = id;
        this.name = name;
        this.addr = addr;
        this.phone = phone;
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        builder.append("id=");
        builder.append(id);

        builder.append(" name=");
        builder.append(name);

        builder.append(" addr=");
        builder.append(addr);

        builder.append(" phone=");
        builder.append(phone);
        return builder.toString();
    }
}

```
如上代码所示，tableName表示自定义表名，默认为类名，primarykey声明主键，可单键也可多键，indices表示表索引，可索引，ColumnInfo表示自定义列名，默认为变量名，ignore表示不建此列。
注意，若field不为public则必须添加getter和setter方法。public简单省事。
# 上代码

## Dao

dao必须为接口或者抽象类
```java
@Dao
public interface UserDao {


    @Query("select * from users where name like :name")
    Flowable<List<User>> getUsers(String name);
    @Query("select * from users ")
    Flowable<List<User>> getAllUsers();

   @Insert
    long insert(User user);

   @Delete
   int delete(User user);

   @Update
   int updateUser(User user);

   @Update
   int updateUser(List<User> users);

}
```
方法名自定义，返回值这四种有区别Update,delete返回int表示操作的行数，而insert返回的是行号所以是long，另外这几个都支持rxjava，所以可以直接返回flowable.

因为对数据 库的操作最终都是通过dao进行的。所以增(insert)删(delete)改(update)查(query)都写上。
除了query接受sql语句外，其他都不接受sql语句，自动根据传入的entity中的primarykey去进行相应的操作。所以问题就来了，如果我想模糊批量进行update或者delete怎么办呢？使用query,在query中写相应的sql语句，如果需要引用参数，使用':paramName'。
Update,delete,Insert即可以接受单个实体也可以接受多个entity,多entity可以用java的可变参数"User ...user"或传list


## Database
新建一个class名字叫UserDB，必须为抽象类，这个类名跟数据库名字没半毛钱关系，后续会提到数据库名字
```java
@Database(entities = User.class,version = 1,exportSchema = false)
public abstract class UserDB extends RoomDatabase {
    public abstract UserDao getUserDao();
}
```
entities声明包含哪些实体(即表)， version,当前的版本. 
其中的抽象方法为声明获取dao的获取，定义抽象方法即可，自动生成。

## 使用datase建立数据库
```java
UserDB userdb= Room.databaseBuilder(this,UserDB.class,"users").build();
```

实例化数据库比较耗资源，强烈建议在application中实例化，使用时引用application中的实例保持单例。


## 简单地使用一下

```java
	UserDB   db = ((CusApp)getApplication()).userdb;
    UserDao    dao = db.getUserDao();
        Flowable.range(0,20).map(res ->{

            User user = new User(0, "2sample"+res, "LA" ,res+"phone"+res );
            long size = dao.insert(user) ;
            Log.i(TAG, "size: " +size);
            return size;
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(every -> {count = count+ every;
                    Log.i(TAG, "onCreate: " +count);});

        
       dao.getAllUsers().flatMap(list -> Flowable.fromIterable(list))
               .subscribeOn(Schedulers.io())
               .observeOn(AndroidSchedulers.mainThread())
               .subscribe(user ->  {user.id = 7989; Log.i(TAG, "user : " + user.toString());});
```

# 完结

增删改查必须不能在主线程，但是如果返回的是flowable例外，这个真是太棒了。

入门就到这了。另外还有Relationships，nested entity,type converters,Database migration期待精通。
