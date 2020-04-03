### Builder Pattern   



1. 适合参数很多的对象
2. 或者存在必选参数，或可选参数
3. 注意：Builder是非线程安全的，所以如果我们需要判断对象的某些属性，例如年龄等，需要在`build`方法中先创建`User`对象，然后在判断


```java
public class User {
	
	private String firstName;
	
	private String secondName;
	
	private int age;
	
	private String gender;
	
	private User(UserBuilder builder) {
		this.firstName = builder.firstName;
		this.secondName = builder.secondName;
		this.age = builder.age;
		this.gender = builder.gender;
	}
	
	public String getFirstName() {
		return firstName;
	}
	
	public String getSecondName() {
		return secondName;
	}
	
	public int getAge() {
		return age;
	}
	
	public String getGender() {
		return gender;
	}
	
	
	public static class UserBuilder {
		
		private String firstName;
		
		private String secondName;
		
		private int age;
		
		private String gender;
		
		public UserBuilder(String firstName, String secondName) {
			this.firstName = firstName;
			this.secondName = secondName;
		}
		
		public UserBuilder firstName(String firstName) {
			this.firstName = firstName;
			return this;
		}
		
		public UserBuilder secondName(String secondName) {
			this.secondName = secondName;
			return this;
		}
		
		public UserBuilder age(int age) {
			this.age = age;
			return this;
		}
		
		public UserBuilder gender(String gender) {
			this.gender = gender;
			return this;
		}
		
		public User build() {
			return new User(this);
		}
	}
}
```