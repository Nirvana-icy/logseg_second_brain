- 在Swift中可以定义两种类：
  
  一种是从NSObject或者派生类派生的类
  
  一种是从系统Swift基类SwiftObject派生的类
- Swift语言中类方法：
  
  1. OC类的派生类并且重写了基类的方法
  2. 类中定义的常规方法 
  3. extension中定义的方法
  
  ```Swift源代码
  
   //类定义
  class MyUIView:UIView {
      open func foo(){}   //常规方法
      override func layoutSubviews() {}  //重写OC方法
  }
  
  func main(){
    let obj = MyUIView()
    obj.layoutSubviews()   //调用OC类重写的方法
    obj.foo()   //调用常规的方法。
  }
  ```
  
  ```C伪代码
  //...........................................运行时定义部分
  
  //OC类的方法结构体
  struct method_t {
      SEL name;
      IMP imp;
  };
  
  //Swift类描述
  struct swift_class {
      ...   //其他的属性，因为这里不关心就不列出了。
      struct method_t  methods[1];
      ...   //其他的属性，因为这里不关心就不列出了。
      //虚函数表刚好在结构体的第0x50的偏移位置。
      IMP vtable[1];
  };
  
  
  //...........................................源代码中类的定义和方法的定义和实现部分
  
  //类定义
  struct MyUIView {
        struct swift_class *isa;
  }
  
  //类的方法函数的实现
  void layoutSubviews(id self, SEL _cmd){}
  void foo(){}  //Swift类的常规方法中和源代码的参数保持一致。
  
  //类的描述信息构建，这些都是在编译代码时就明确了并且保存在数据段中。
  struct swift_class classMyUIView;
  classMyUIView.methods[0] = {"layoutSubviews", &layoutSubviews};
  classMyUIView.vtable[0] = {&foo};
  
  
  //...........................................源代码中程序运行的部分
  
  void main(){
    MyUIView *obj = MyUIView.__allocating_init(classMyUIView);
    obj->isa = &classMyUIView;
    //OC类重写的方法layoutSubviews调用还是用objc_msgSend来实现
    objc_msgSend(obj, @selector(layoutSubviews);
    //Swift方法调用时对象参数被放到x20寄存器中
    asm("mov x20, obj");
    //Swift的方法foo调用采用间接调用实现
    obj->isa->vtable[0]();
  }
  ```
  
  ```extension中定义的方法
  ////////Swift源代码
  
  //类定义
  class CA {
      open func foo(){}
  }
  
  //类的extension定义
  extension CA {
     open func extfoo(){}
  }
  
  func main() {
    let obj = CA()
    obj.foo()
    obj.extfoo()
  }
  ```
  ```
  ////////C伪代码
  
  //...........................................运行时定义部分
  
  
  //Swift类描述。
  struct  swift_class {
      ...   //其他的属性，因为这里不关心就不列出了。
     //虚函数表刚好在结构体的第0x50的偏移位置。
      IMP vtable[1];
  };
  
  
  //...........................................源代码中类的定义和方法的定义和实现部分
  
  
  //类定义
  struct CA {
        struct  swift_class *isa;
  }
  
  //类的方法函数的实现定义
  void foo(){}
  //类的extension的方法函数实现定义
  void extfoo(){}
  
  //类的描述信息构建，这些都是在编译代码时就明确了并且保存在数据段中。
  //extension中定义的函数不会保存到虚函数表中。
  struct swift_class classCA;
  classCA.vtable[0] = {&foo};
  
  
  //...........................................源代码中程序运行的部分
  
  void main(){
    CA *obj =  CA.__allocating_init(classCA)
    obj->isa = &classCA;
    asm("mov x20, obj");
    //Swift中常规方法foo调用采用间接调用实现
    obj->isa->vtable[0]();
    //Swift中extension方法extfoo调用直接硬编码调用，而不是间接调用实现
    extfoo();
  }
  
  ```
- Swift类中成员变量的访问