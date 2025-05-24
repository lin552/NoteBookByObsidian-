---
创建时间: 2025-04-16 11:56:42
作者: wangxiaoming
tags:
  - Parcelable
  - Serializable
---
#### 一、本质与设计初衷
- ​**`Serializable`**​（Java 原生）
    - ​**定义**​：Java 提供的通用序列化接口，通过实现 `java.io.Serializable` 即可标记对象可序列化
    - ​**特点**​：
        - 简单易用，仅需声明接口，无需额外代码（如 `serialVersionUID` 可省略，但建议显式定义）
        - 依赖反射机制，序列化/反序列化过程产生临时对象，性能较低（内存占用高，速度慢）
        - 支持数据持久化（存储到文件/数据库）和网络传输，兼容性好
- ​**`Parcelable`**​（Android 特有）
    - ​**定义**​：专为 Android 内存高效传输设计的序列化接口，需手动实现方法
    - ​**特点**​：
        - 性能优势显著（内存操作，速度比 Serializable 快 10 倍以上）
        - 实现复杂（需重写 `writeToParcel`、`describeContents` 及静态 `CREATOR`）
        - 仅适用于内存数据传输，不推荐持久化（不同 Android 版本可能不兼容）
#### 使用方法对比
##### 1）`Serializable`
```java
//实现接口  
public class User implements Serializable {  
    //显式定义 serialVersionUID (避免类结构变化导致反序列化失败)  
    private static final long serialVersionUID = 1L;  
    private String name;  
    private transient int age; //transient 修饰的字段不参与序列化  
  
    public static void main(String[] args) {  
        // 序列化与反序列化  
        User user = new User();  
        ObjectOutputStream.writeObject(user);  
        User newUser = (User) ObjectOutputStream.readObject();  
    }  
}
```
- 扩展功能：支持自定义序列化逻辑（重写 `writeObjcet/readObject`）
##### 2）`Parcelable`
```java
public class User implements Parcelable {
   private String name;
   private int age;
   //重写序列化方法
   @Override
   public void writeToParcel(Parcel dest,int flags){
      dest.writeString(name);
      dest.writeInt(age);
   }
   //必须定义CREATOR
   public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in){
            return new User(in);
        }
        @Override
        public User[] newArray(int size){
            return new User[size];
        }
   }
   //反序列化构造参数
   protected User(Parcel in){
        name = in.readString();
        age = in.readInt();
   }
}
```
工具优化：Android Studio 可通过插件（如 `Parcelable Code Generator`）自动生成代码

#### 三、面试高频考点
1. ​**性能差异**​
    - `Parcelable` 直接操作内存，避免反射和 IO 开销，适合高频数据传输（如 Activity 间传对象）
    - `Serializable` 因反射机制，性能较低，但适合低频场景（如存储配置信息）
2. ​**实现复杂度**​
    - `Parcelable` 需手动处理字段顺序和类型匹配（写入与读取顺序必须一致）
3. ​**兼容性与安全性**​
    - Serializable 支持跨平台和版本兼容，但数据可能被篡改；`Parcelable` 仅限 Android，但内存操作更安全
4. ​**特殊场景**​
    - ​**transient 关键字**​：Serializable 中标记字段不参与序列化
    - ​**List/Map 处理**​：`Parcelable` 需用 `writeList`/`readList` 等方法

#### 四、适用场景与选择建议
| ​**场景**​               | ​**推荐方案**​          | ​**理由**​                                                    |
| ---------------------- | ------------------- | ----------------------------------------------------------- |
| Activity/Fragment 间传对象 | Parcelable          | 内存操作高效，避免 `TransactionTooLargeException`（Bundle 大小限制 1-2MB） |
| 数据持久化（存储到文件/数据库）       | Serializable        | 兼容性强，支持长期存储                                                 |
| 跨进程通信（AIDL）            | Parcelable          | 通过 Binder 传递，性能优先                                           |
| 网络传输或复杂对象存储            | Serializable 或 JSON | 兼容性要求高，且需持久化（如 Retrofit 结合 `GSON`）                          |
#### 五、总结
- ​**优先选 `Parcelable`**​：Android 内部高频数据传输（如界面跳转、进程通信）。
- ​**选 `Serializable`**​：需持久化或跨平台场景（如与后端 Java 服务交互）。
- ​**优化技巧**​：
    - 使用 `Parcelable` 插件减少手写代码量
    - 避免在 `Parcelable` 中传递过大数据（分页加载或懒加载）