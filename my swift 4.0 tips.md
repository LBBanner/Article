##一. 类型转换as? as! as
###as?
* 前面的结果是可选的
* if let / guard let一定过要用as?，也只有可选类型才能用if let / guard let解包

###as！
* 前面的结果一定有值

###as
* NSString -> String
* NSArray -> Array / []
* NSDictionary -> Dictionary / [:]
* 因为底层做了结构体和OC对象的桥接

###什么时候需要类型转换？
* 将父类转换为子类
* 因为子类的属性和方法比父类多
* oc中要用(CustomTableViewCell*)tableView dequeueXXX
* 转换有风险，如果没有对应的属性和方法会崩溃


##二. 懒加载

* 保证控件的延迟创建
* 避免开发中处理解包的问题
* 本质是一个闭包
* swift中懒加载只会调用1次，只会在第一次调用的时候创建对象  
oc中如果将懒加载属性置为nil，则会调用2次甚至更多（注意swift中nil是可选类型才有的，nil是一种类型而不是置空）

###swift
不用闭包的情形，自己体会下 ？ 和 ！  
那么会出现各种 ？和 ！如下：  

```swift
class ViewController: UIViewController {

    private var label: UILabel?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        label = UILabel()
        label?.text = "label"
        label?.center = view.center
        label?.sizeToFit()
        view.addSubview(label!)
    }
}
```

写法 一  
懒加载仅仅是实例化，设置属性什么的放在其他地方

```swift
class ViewController: UIViewController {

    private lazy var label = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        label.text = "label"
        label.center = view.center
        label.sizeToFit()
        view.addSubview(label)
    }
}
```
写法二  
闭包缩略形式的懒加载

```swift
class ViewController: UIViewController {

    private lazy var label: UILabel = {
        let label = UILabel()
        label.text = "label"
        return label
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.addSubview(label)
    }
}
```
写法三  
闭包完整形式的懒加载

```swift
class ViewController: UIViewController {

    private lazy var label = { () -> UILabel in
        let label = UILabel()
        label.text = "label"
        return label
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.addSubview(label)
    }
}

```
###OC

```objc
#import "ViewController.h"

@interface ViewController ()
@property (nonatomic,strong) UILabel *label;
@end

@implementation ViewController

-(UILabel *)label {
    if (_label == nil) {
        _label = [[UILabel alloc] init];
        _label.text = @"test";
        _label.center = self.view.center;
    }
     return _label;
}
- (void)viewDidLoad {
    [super viewDidLoad];
    // 懒加载第1次会创建label
    [self.view addSubview:self.label];
    // 这里释放label
    _label = nil;
    // 因为前面释放了label，label为nil，所以这里懒加载第2次会创建label
    NSLog(@"%@",self.label);
}

@end
```

##三. getter setter

swift中一般不会同时重写setter和getter方法，很繁琐，好像也没有应用场景

```swift
class Person: NSObject {

    // getter & setter,一般开发很少这么用
    private var _name: String?
    
    var name: String? {
        get {
            // getter方法，返回成员变量
            return _name
        }
        set {
            // setter方法，存值
            _name = newValue
        }
    }
}
```

swift中的只读属性

```swift
// 写法一
class Person: NSObject {
// oc定义属性的时候，有一个readonly -> 重写getter方法
    var title: String {
        // 只重写getter方法，不重写setter方法
        // 即swift中的只读属性
        get {
            return "title"
        }
    }
}

// 写法二
class Person: NSObject {
	// 只读属性的简写，直接return
	// 又称为“计算”属性，本身不保存内容，都是通过计算获得结果
	// 类似于一个函数，但是没有参数，一定有返回值
    var title: String {
        return "title"
    }
}

```

### 计算型属性和懒加载的对比

####计算型属性
* 不分配独立的存储空间保存计算结果
* 每次调用都会被执行
* 更像一个函数，不过不能接受参数，同时必须要有返回值

####懒加载属性
* 在第一次调用时，执行闭包并且分配空间储存闭包返回的数值
* 会分配独立的储存空间
* 与OC不同的是，lazy属性即使被设置为nil也不会被再次调用


注意：一个有lazy，一个没有；一个有等号，一个没有；一个最后有括号，一个没有。
