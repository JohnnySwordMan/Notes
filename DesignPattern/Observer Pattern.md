### Observer Pattern   



**定义**：定义对象间一种一对多的关系，使得每当一个对象改变的时候，则所有依赖于它的对象都会得到通知并被自动更新。例如：User状态的改变

除了使用以下示例，还可以使用`RxJava`来订阅监听


```kotlin
interface Observer {
	
	fun update()
}

interface Observable {
	
	fun addObserver(observer: Observer)
	
	fun removeObserver(observer: Observer)
	
	fun notifyObservers()

}


// 定义一个Observer
class PublishObserver : Observer{
	
	fun update() {
		// 发布视频
	}
}

// 定义一个Observable
class PublishObservable : Observable {

	private val arr = MutableList<Observer>()

	fun addObserver(observer: Observer) {
		arr.add(observer)
	}
	
	fun removeObserver(observer: Observer) {
		arr.remove(observer)
	]
	
	fun notifyObsevers() {
		for(observer: arr) {
			observer.update()
		}
	}

}
```