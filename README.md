# C-calculator-interface
现一个能进行实数间加、减、乘、除运算的简易计算器。首先创建一个基于 QWidget 带界面的 Qt 项目，然后按照如下步骤进行操作：

# 01、计算器界面设计
在界面中拖入两个单行文本框和十七个按钮，按钮上显示的文字、按钮对象和单行文本框对象名如图 1 所示。为了美观起见，设置窗口为“栅格布局”以对齐部件。

将窗口对象的“windowTitle”属性设置为“计算器”；勾选掉第一个单行文本框（lineEdit_Show 对象）的“enable”属性，使得该单行文本框变为灰色（该文本框仅用于显示结果）；勾选上第二个单行文本框（lineEdit_Input 对象）的“readOnly”属性（限制用户不能直接在文本框中通过键盘输入内容）、将其的“alignment”水平属性设置为“ALignRight”。




■ 图 1 计算器界面设计

# 02、计算器功能实现
进行算术运算时，之前输入的左操作数和操作符需存储（右操作数可直接从文本框 lineEdit_Input 中读取），因此在自定义窗口 Widget 类中添加私有数据成员如下：

     QString operandStr1; //用于存储字符串形式的左操作数

     QString operatorStr; //用于存储操作符

并在 Widget 类的构造函数体中添加如下语句以将它们初始化为空串：

     operandStr1="";

     operatorStr="";
接下来实现点击各个按钮时触发的功能：给每个按钮的 clicked()信号都添加自关联槽。以按钮 1 为例，自关联槽实现代码如下：


     void Widget::on_btn_1_clicked()

     {

       ui->lineEdit_Input->setText(ui->lineEdit_Input->text()+"1");

     }
按钮 2 到 9 的功能实现和按钮 1 是类似的，只需把上述代码中的字符"1"改成对应的数字字符即可。对于按钮 0，由于一般不会出现诸如“00”形式的数字“0”，因此代码中对这种情况进行了处理，自关联槽定义如下：

     void Widget::on_btn_0_clicked()

     {

     if(ui->lineEdit_Input->text()!="0")

     ui->lineEdit_Input->setText(ui->lineEdit_Input->text()+"0");

     }
小数点按钮需要考虑按下时前面没有数字的情形（此时默认为整数部分为 0）、按下时串中已有了小数点的情形，最终自关联槽定义如下：

     void Widget::on_btn_Point_clicked()

     {

     if(ui->lineEdit_Input->text()=="")

     ui->lineEdit_Input->setText("0.")
     ;
     else if(ui->lineEdit_Input->text().contains(".")==true)

     ; //数字串中已有小数点，不能再输入

     else

     ui->lineEdit_Input->setText(ui->lineEdit_Input->text()+".");
     }

clear 按钮点击时，只需将文本框 lineEdit_Input 中的内容清空即可，代码如下：


     void Widget::on_btn_clear_clicked()

     {

     ui->lineEdit_Input->clear();

     }

加、减、乘、除按钮的实现是类似的。以加法按钮为例：分情况进行处理，如果按下时文本框中 lineEdit_Input 是空串则不进行任何处理直接结束；否则说明用户提供了一个操作数，接下来判断它是左操作数还是右操作数；若 operandStr1 为空说明文本框中是左操作数，将其存储到 operandStr1、将运算符“+”存储到 operatorStr、将文本框清空以待用户再次输入右操作数、将已输入的内容显示于文本框 lineEdit_Show 中；若 operandStr1 不为空说明文本框中已是右操作数，此时按下加号和按下等号作用是相同的，直接调用点击等号按钮关联的槽函数 on_btn_Calc_clicked 进行处理即可。实现代码如下：

     void Widget::on_btn_Add_clicked()

     {

     if(ui->lineEdit_Input->text()=="") //没有输入数据

     return;

     else if(operandStr1=="") //没有左操作数

     {

     operandStr1=ui->lineEdit_Input->text();

     operatorStr="+";

     ui->lineEdit_Input->clear();//输入文本框清空，以待输右操作数

     ui->lineEdit_Show->setText(operandStr1+operatorStr);
     
     }

     else

     on_btn_Calc_clicked();

     }

其它三个运算符的实现是一样的，只需将上述代码中赋值给 operatorStr 的字符串改成相应的运算符即可。

最后是等号按钮的实现，代码如下：

     void Widget::on_btn_Calc_clicked()

       {
  
     if(operandStr1!=""&&ui->lineEdit_Input->text()!=""&&operatorStr!="")

     {

       double result;
  
       double operand1=operandStr1.toDouble();
  
       double operand2=ui->lineEdit_Input->text().toDouble();
  
       if(operatorStr=="+")
  
       result=operand1+operand2;
  
       else if(operatorStr=="-")
  
       result=operand1-operand2;
  
       else if(operatorStr=="*")
  
       result=operand1*operand2;
  
       else if(operatorStr=="/")
  
       if(operand2!=0.0)
  
       result=operand1/operand2;
  
       else
  
       {
  
       QMessageBox::warning(this,"提示","除数不能为零");
  
       result=0;
  
       }
  
       ui->lineEdit_Show->setText(QString::number(result));
  
       operandStr1=""; //计算完毕，操作数清空
       
       operatorStr=""; //计算完毕，操作符清空
       
       ui->lineEdit_Input->clear(); //计算完毕，数据输入文本框清空
  
         }     
    
     }



首先判断左右操作数和运算符都存在，然后将 QString 字符串形式的操作数转换为 double 类型的操作数，由 QString 类的成员函数 toDouble 实现；然后再根据运算符的不同分别进行不同的计算，结果放在 result 中；对于除法，代码中添加了对除数为 0 时的出错处理；最后将算得的结果显示在文本框 lineEdit_Show 中（QString 的静态成员函数 number 用于将给定的参数转换为字符串形式），并清空相关数据以等待下一次计算。

上述代码中使用到了 QMessageBox 类，因此还需在 widget.cpp 文件中添加头文件：

#include<QMessageBox>
到此功能已全部实现，运行程序可查看效果。图 3-33 的左图为输入了按下按钮“2”、“3”、“+”之后的效果，右图为再按下按钮“5”、“.”、“1”、“=”之后的效果。




■ 图 2 计算器运行效果示例

# 03、登陆界面设计
接下来，考虑在上述已实现计算器的基础上添加一个登陆的功能：程序运行时首先显示一个登陆界面，只有当输入了正确的用户名和密码后，才能打开计算器。

实际上，除了可以在项目创建向导中给应用程序主窗口选择“使用界面”外，在项目中新增自定义 C++窗口（或部件）类时，也可以使用界面。接下来在项目中添加一个带界面的登陆对话框类，操作步骤如下：

在项目名处右单击，弹出的菜单中选择“AddNew…”，打开如图 3 所示的界面。选中“Qt”下的 Qt 设计师界面类，以创建一个 Qt 设计师界面类。




■ 图 3 选择创建 Qt 设计师界面类

点击图 3 的 “Choose…”按钮后进入到选择界面模板的步骤，见图 4。所谓“选择界面模板”，是指准备在哪种窗口（或部件）的基础上进行更多的设计（即选择窗口或部件类型做为自定义窗口类的基类），本文中选择“DialogwithoutButtons”，即使用 QDialog 类作为基类。




 ■ 图 4 选择界面模板

点击“下一步”，进入图 5 所示界面。在类名文本框中填入自定义登陆对话框类的名字（本文使用“LoginDialog”），下方会自动生成相关的头文件、源文件和界面文件的名字，也可以修改这些文件的文件名（但不建议修改）以及默认以当前工程目录为存放路径。




■ 图 5 设置自定义登陆对话框类的类名

再次点击“下一步”，完成带界面自定义登陆对话框类的初始创建。可以看到，工程中已新增了三个与该类有关的文件。

双击“logindialog.ui”文件打开界面设计师。将对话框窗口标题（windowTitle 属性）设置为“登陆”；然后往窗口中拖入两个标签、一个按钮和两个单行文本框；标签和按钮显示的文字如图 6 所示，各部件的名字见图中的对象浏览器窗口。




■ 图 6 登陆对话框界面设计

设置标签为右对齐(alignment 属性:“AlignRight”)；设置密码文本框的 echoMode 为“Password”，使得输入数据时显示为表示密码的黑色圆点；将窗口设置为“栅格布局”以方便的对齐各个部件；为了方便用户使用，还可指定部件的 Tab 键顺序、设置标签的快捷键及和单行文本框的伙伴关系。

# 04、登陆功能的实现
首先给登陆对话框类添加一个信号，在类定义中(logindialog.h 文件)添加代码如下：

     signals:

     void LoggedIn();

然后给“登陆”按钮的 clicked()信号添加自关联槽，代码实现如下：


     void LoginDialog::on_loginBtn_clicked()

     {
     if(ui->userEdit->text()=="admin"&&ui->pwdEdit->text()=="123456")

     {

     emit(LoggedIn());

     hide(); //隐藏登陆窗口

     }

     else

     QMessageBox::information(this, "提示",

                                 "用户名和密码错误");
                            
     }




当输入了正确的用户名和密码时将发射 LoggedIn 信号（目的是通知计算器窗口把自己显示出来，信号槽关联在随后的主函数中），然后将登录对话框隐藏；否则提示用户名和密码出错。

在 logindialog.cpp 文件中还需添加如下头文件：

     #include<QMessageBox>

为了实现先显示登陆界面，成功后再显示计算器界面的操作，以及将登陆对话框发射的 LoggedIn 信号和计算器窗口的显示槽函数 show 进行关联，主函数也需要进行修改，代码如下:

     #include "widget.h"

     #include <QApplication>

     #include "logindialog.h"

 
     int main(int argc, char *argv[])

     {

         QApplication a(argc, argv);
    
 
     Widgetw;

         LoginDialog login(&w);
    
         login.show();
    
         QObject::connect(&login,SIGNAL(LoggedIn()),&w,SLOT(show()));
    
 
         return a.exec();
     }


运行时，首先会显示出图 7 所示的登陆界面。在输入正确的用户名和密码、并按下登陆按钮后，才打开图 1 的界面。




■ 图 7 登陆对话框运行效果
