# 4   MFC 工具栏及两大机制



## 4.1   工具栏

### 4.1.1   工具栏相关类

- `CToolBarCtrl` ：其父类为 `CWnd` ，封装了关于工具栏控件的各种操作（该类的对象不代表工具栏，它代表工具栏里的按钮）
- `CToolBar` ：其父类为 `CContorlBar` ，封装了关于工具栏的操作，以及和框架窗口的关系（该类的对象代表工具栏）；工具栏与框架窗口之间的关系为父子窗口和停靠关系

### 4.1.2   工具栏构建步骤

1. 添加工具栏资源（操作可视化界面即可完成）

   参考4.1.3.2节

2. 创建工具栏

   - `CToolBar::CreateEx`

3. 加载工具栏的资源
   - `CToolBar::LoadToolBar`

4. 设置工具栏的停靠（船坞化）
   - 船坞化：将「工具栏 `CToolBar` 」类比为「船」，将「框架窗口 `CFrameWnd` 」类比为「港口」
   - 首先，应在 `CToolBar::CreateEx` 中添加 `CBRS_GRIPPER` 这一参数（夹子/把手风格），工具栏左侧的移动把手就出现了
   - `CToolBar::EnableDocking`：「船（工具栏）」想停靠在哪里？
   - `CFrameWnd::EnableDocking`：「港口（框架窗口）」允许「船（工具栏）」停靠在哪里？（与上一个必须要有交集）
   - `CFrameWnd::DockControlBar`：「港口（框架窗口）」设置「船（工具栏）」临时停在哪里，需指明是哪个「船（工具栏）」可以停靠在「港口（框架窗口）」的哪里

### 4.1.3   工具栏构建示例

#### 4.1.3.1   新建菜单

**「工具栏」**通常与**「菜单」**绑定使用，即：在「工具栏」中的功能，在「菜单」中仍然有；但其实在原理上二者并无关系，它们能够绑定的原因在于：在设计时，工具栏按钮被鼠标点击之后其触发的消息与使用鼠标点击对应的某个菜单项触发的消息相同（均设置为相同的 `WM_COMMOND` 消息）。因此，我们先新建一个最简单的菜单，后面学习如何将两者绑定：

1. **新建资源文件**

   在「解决方案视图」中，右击项目名称，选择添加一新建项，选择「资源」→「资源文件」，将其命名为「MFCToolBar.rc」

   ![](.\assets\ToolBar演示添加资源文件-1696939313738-11.png)

2. **添加菜单资源**

   在「资源视图」中，右击资源文件「MFCToolBar.rc」→ 选择「添加资源」→ 选择「Menu」

   ![](.\assets\右键点击rc文件添加工具栏资源-1696940695190-16.png)

   ![](.\assets\ToolBar演示添加菜单栏-1696939299312-9.png)

3. **新建最基础的菜单项**

   与第三章相同，双击资源视图中的菜单栏ID「IDR_MENU1」，在主菜单栏下新建一子菜单项「新建」，然后修改其ID为「ID_NEW」

   ![](.\assets\ToolBar演示可视化编辑菜单栏图片-1696940296264-14.png)

4. **编写代码：将菜单挂载在窗口上、处理「新建」所发出的消息**

```c++
/**
 * 项目名称: MFCToolBar
 * 项目设置: win32 + Windows application + MFC
 * 课程示例: MFC工具栏 - 菜单挂载
*/

#include <afxwin.h>
#include "resource.h"

class CMyFrameWnd : public CFrameWnd {
    DECLARE_MESSAGE_MAP()
public:
    afx_msg void OnNew() {
        AfxMessageBox("新建菜单项被点击");
    }
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
    ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()

class CMyWinApp : public CWinApp {
public:
    virtual BOOL InitInstance() {
        CMyFrameWnd* pFrame = new CMyFrameWnd;
        pFrame->Creat(
        	NULL,
            "MFCToolBar",
            WS_OVERLAPPEDWINDOW,  	// 窗口风格
            CFrameWnd::rectDefault, // 主框架窗口大小（宽/高）
            NULL,					// 副窗口（没有）
            (CHAR*)IDR_MENU1		// 菜单：将菜单挂载在窗口上
        );
        m_pMainWnd = pFrame;
        pFrame->ShowWindow(SW_SHOW);
        pFrame->UpdateWindow();
        return TRUE;
    }
};

CMyWinApp theApp;
```

==点击菜单项所触发的消息类型为 `ON_COMMAND` ，为了区分不同的菜单项需要使用不同的消息ID==，因此需要在代码中处理相应的 `ON_COMMAND` 消息，消息ID上一步更改为 `ID_NEW` ，消息处理函数为 `OnNew` 

#### 4.1.3.2   构建工具栏

1. **在 `.rc` 文件中添加工具栏资源**

   在「资源视图」中，右击资源文件「MFCToolBar.rc」→ 选择「添加资源」→ 选择「Toolbar」

   ![](.\assets\选择Toolbar添加工具栏资源-1696940776807-18.png)

2. **编辑工具栏资源**

   - 【绘制】而后在画布上即可绘制工具栏样式
   - 【设置】右击工具按钮，点击属性，即可设置该工具按钮的ID，只需要将其设置的与上面添加的某个菜单栏选项的ID相同就可以复用消息处理函数（即本节开头所述的 “绑定” ）
   - 【删除】鼠标左键按住某个按钮往外拽可以删除该按钮

3. **编写代码：创建并加载工具栏资源**

   框架窗口与工具栏存在父子关系，通常，子窗口的创建在父窗口处理 `WM_CREATE` 消息的函数中创建，因此我们应==在处理框架窗口的 `WM_CREATE` 消息时创建工具栏==

   **关键内容总结：**

   1. 在处理框架窗口的 `WM_CREATE` 消息时创建工具栏，则需要将宏 `ON_WM_CREATE()` 放在实现宏内，并且编写 `afx_msg int OnCreate(LPCREATESTRUCT pcs)` 函数来处理 `WM_CREATE` 消息，在函数内部创建并加载工具栏资源
   2. 使用 `CToolBar::CreateEx` 创建工具栏资源，使用 `CToolBar::LoadToolBar` 加载工具栏资源
   3. 通常将需要长时间存在的子窗口作为父窗口的类内成员变量
   4. 工具栏的类 `CToolBar` 为扩展类，因此需要头文件包含：`#include <afxext.h>`

```c++
/**
 * 项目名称: MFCToolBar
 * 项目设置: win32 + Windows application + MFC
 * 课程示例: MFC工具栏 - 管理工具栏
*/

#include <afxwin.h>
#include "resource.h"
#include <afxext.h>

class CMyFrameWnd : public CFrameWnd {
    DECLARE_MESSAGE_MAP()
public:
    afx_msg void OnNew() {
        AfxMessageBox("新建菜单项被点击");
    }
    afx_msg int OnCreate(LPCREATESTRUCT pcs) {
        this->toolbar.CreateEx(  				// 工具类对象调用CreateEx以创建工具栏
        	this,  								// 父窗口的对象：pFrame
            TBSTYLE_FLAT,  						// 工具栏按钮风格（平滑风格）
            WS_CHILD|WS_VISIBLE|CBRS_ALIGN_TOP, // 工具栏容器的风格（子窗口|最初可见|停靠于上部）
        );
        this->toolbar.LoadToolBar(IDR_TOOLBAR1);// 加载工具栏资源
        return CFrameWnd::OnCreate(pcs);
    }
public:
    CToolBar toolbar;  // 工具类对象  // #include <afxext.h>
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
    ON_COMMAND(ID_NEW, OnNew)
    ON_WM_CREATE()
END_MESSAGE_MAP()

class CMyWinApp : public CWinApp {
public:
    virtual BOOL InitInstance() {
        CMyFrameWnd* pFrame = new CMyFrameWnd;
        pFrame->Creat(
        	NULL,
            "MFCToolBar",
            WS_OVERLAPPEDWINDOW,  	// 窗口风格
            CFrameWnd::rectDefault, // 主框架窗口大小（宽/高）
            NULL,					// 副窗口（没有）
            (CHAR*)IDR_MENU1		// 菜单：将菜单挂载在窗口上
        );
        m_pMainWnd = pFrame;
        pFrame->ShowWindow(SW_SHOW);
        pFrame->UpdateWindow();
        return TRUE;
    }
};

CMyWinApp theApp;
```

#### 4.1.3.3   工具栏的停靠/船坞化

```c++
/**
 * 项目名称: MFCToolBar
 * 项目设置: win32 + Windows application + MFC
 * 课程示例: MFC工具栏 - 管理工具栏 - 工具栏的停靠/船坞化
*/

#include <afxwin.h>
#include "resource.h"
#include <afxext.h>

class CMyFrameWnd : public CFrameWnd {
    DECLARE_MESSAGE_MAP()
public:
    afx_msg void OnNew() {
        AfxMessageBox("新建菜单项被点击");
    }
    afx_msg int OnCreate(LPCREATESTRUCT pcs) {
        // 工具栏的新建与加载
        this->toolbar.CreateEx(  				  // 工具类对象调用CreateEx以创建工具栏
        	this,  								  // 父窗口的对象：pFrame
            TBSTYLE_FLAT,  						  // 工具栏按钮风格
            WS_CHILD|WS_VISIBLE|CBRS_ALIGN_TOP|CBRS_GRIPPER,   // 工具栏容器的风格
        );
        this->toolbar.LoadToolBar(IDR_TOOLBAR1);  // 加载工具栏资源
        
        // 工具栏的船坞化
        this->toolbar.EnableDocking(CBRS_ALIGN_ANY);  // 工具栏哪里都想停靠（任意位置）
        this->EnableDocking(CBRS_ALIGN_ANY);  // 框架类觉得工具栏停哪都行（任意位置）
        this->DockControlBar(&toolbar, AFX_IDW_DOCKBAR_TOP);  // 使toolbar停在最上面
        return CFrameWnd::OnCreate(pcs);
    }
public:
    CToolBar toolbar;  // 工具类对象  // #include <afxext.h>
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
    ON_COMMAND(ID_NEW, OnNew)
    ON_WM_CREATE()
END_MESSAGE_MAP()

class CMyWinApp : public CWinApp {
public:
    virtual BOOL InitInstance() {
        CMyFrameWnd* pFrame = new CMyFrameWnd;
        pFrame->Create(
        	NULL,
            "MFCToolBar",
            WS_OVERLAPPEDWINDOW,  	// 窗口风格
            CFrameWnd::rectDefault, // 主框架窗口大小（宽/高）
            NULL,					// 副窗口（没有）
            (CHAR*)IDR_MENU1		// 菜单：将菜单挂载在窗口上
        );
        m_pMainWnd = pFrame;
        pFrame->ShowWindow(SW_SHOW);
        pFrame->UpdateWindow();
        return TRUE;
    }
};

CMyWinApp theApp;
```

#### 4.1.3.4   总结

- 工具栏的使用流程可参考 4.1.2 节
- 「菜单」和「工具栏」其实都是「主框架窗口」的子窗口
  - 对于「菜单」，第 3 节中提到将菜单设置/挂到窗口上的其中一种方法是在框架类 `CMyFrameWnd` 内构造 `CMenu` 对象并进行加载等操作
  - 对于「工具栏」，其使用方法与「菜单」相似，是在框架类 `CMyFrameWnd` 内构造 `CToolBar` 对象再进行后续操作
  - 这两种方法在逻辑上都可视为：「主框架窗口」包含「菜单」和「工具栏」，因此类的关系也可以包含



## 4.2 运行时类信息机制

> 回顾之前已经学过：「程序启动机制」、「窗口创建机制」、「消息映射机制」，本节将介绍第四大机制——「运行时类信息机制」
>
> ==**「运行时类信息机制」的作用是：在程序运行过程中可以获知对象的类的相关信息**==，例如：对象是否属于某个类

### 4.2.1 运行时类信息机制的使用

当一个类具备下列三个要素后，==`CObject::IsKindOf` 函数就可以正确判断对象是否属于某个类==：

- 类必须派生自 `CObject`，直接派生或间接派生均可
- 类内必须添加声明宏 `DECLARE_DYNAMIC( theClass )` ，`theClass` 为本类类名
- 类外必须添加实现宏 `IMPLEMENT_DYNAMIC( theClass, baseClass )`，`baseClass` 为父类类名

示例代码如下：

- 定义一个 `CAnimal` 类，派生自 `CObject` 类，类内添加声明宏、类外添加实现宏
- 定义一个 `CDog` 类，派生自 `CAnimal` 类，类内添加声明宏、类外添加实现宏

```c++
#include <afxwin.h>
#include <iostream>

using namespace std;

class CAnimal : public CObject {
	DECLARE_DYNAMIC(CAnimal)
};
IMPLEMENT_DYNAMIC(CAnimal, CObject)

class CDog : public CAnimal {
	DECLARE_DYNAMIC(CDog)
};
IMPLEMENT_DYNAMIC( CDog, CAnimal )

int main() {
	CDog dog;
	cout << "dog is CDog	: "	<< dog.IsKindOf(RUNTIME_CLASS(CDog))	<< endl;	// 1
	cout << "dog is CAnimal	: "	<< dog.IsKindOf(RUNTIME_CLASS(CAnimal)) << endl;	// 1
	cout << "dog is CObject	: "	<< dog.IsKindOf(RUNTIME_CLASS(CObject)) << endl;	// 1
	cout << "dog is CWnd	: "	<< dog.IsKindOf(RUNTIME_CLASS(CWnd))	<< endl;	// 0
	return 0;
}
```

### 4.2.2 运行时类信息机制剖析

为深入理解运行时类信息机制，继4.2.1节代码，将上文所提到的声明宏 `DECLARE_DYNAMIC( theClass )` 与实现宏 `IMPLEMENT_DYNAMIC( theClass, baseClass )` 在 `CDog` 类中展开

#### 4.2.2.1 结构体 `CRuntimeClass`

首先，展开后频繁出现的结构体 `CRuntimeClass` 详细信息如下，可配合后续代码查看：

```c++
struct CRuntimeClass
{
    LPCSTR lpszClassName;	// 【有用】类的名称（char*）
    int m_nObjectSize;		// 【有用】类的大小
    UINT m_wSchema;			// 【没用】类的版本（没有用，所有类的版本均为：0xFFFF）
    CObject* (PASCAL* m_pfnCreateObject)();	// 【当前不用】动态创建机制使用，这里为NULL
    CRuntimeClass* m_pBaseClass;			// 【有用】父类宏展开静态变量地址
	CRuntimeClass* m_pNextClass;			// 【没用】不使用，为NULL
    const AFX_CLASSINIT* m_pClassInit;		// 【没用】不使用，为NULL
}
```

#### 4.2.2.2 宏展开讲解

展开后的 `CDog` 类相关代码如下所示：

```c++
class CAnimal : public CObject {
	DECLARE_DYNAMIC( CAnimal )
};
IMPLEMENT_DYNAMIC( CAnimal, CObject )

class CDog : public CAnimal {
//	DECLARE_DYNAMIC( CDog )  // 将声明宏展开为如下三行有效代码：
public: 
     // class##class_name --> classCDog 在宏中双井号代表“拼接”
	static const CRuntimeClass classCDog;
	virtual CRuntimeClass* GetRuntimeClass() const; 
};

//IMPLEMENT_DYNAMIC( CDog, CAnimal )  // 将实现宏展开：
//IMPLEMENT_RUNTIMECLASS(CDog, CAnimal, 0xFFFF, NULL, NULL)  // 展开第一层依然是宏，继续展开：
AFX_COMDAT const CRuntimeClass CDog::classCDog = { 
		"CDog",	// #class_name --> "CDog" 在宏中井号代表将后面的参数加引号
		sizeof(class CDog), 
		0xFFFF, 
		NULL, 
		((CRuntimeClass*)(&CAnimal::classCAnimal)),  // RUNTIME_CLASS(CAnimal)展开后
		NULL, 
		NULL
};
CRuntimeClass* CDog::GetRuntimeClass() const 
{ 
//	return RUNTIME_CLASS(CDog); 
	return ((CRuntimeClass*)(&CDog::classCDog));
}
```

- **宏展开基本内容**

  声明宏展开后为一个 `CRuntimeClass` 类型的静态公有成员变量 `classCDog` ，和一个公有常虚函数 `GetRuntimeClass()` 的声明；实现宏展开后为静态公有成员 `classCDog` 的类外定义，和公有函数 `GetRuntimeClass()` 的实现

-  **`CRuntimeClass` 类型的公有静态变量**

  以 `classCDog` 为例，重点关注第5个参数 `RUNTIME_CLASS(CAnimal)` ，将宏 `RUNTIME_CLASS` 进一步展开后为 `((CRuntimeClass*)(&CAnimal::classCAnimal))` ，这表明：  `CRuntimeClass` 类型的静态公有成员变量内部保存有本类的父类内部的  `CRuntimeClass` 类型的静态公有成员变量的地址

  即：MFC通过 `CRuntimeClass` 类型的静态公有成员变量，构造了一个链表，最子类为链表的头节点，根据该成员变量即可一路遍历至 `CObject` 类

- **`GetRuntimeClass()` 虚函数**

  该函数返回值原本为 `RUNTIME_CLASS(CDog)` ，展开后为 `((CRuntimeClass*)(&CDog::classCDog))` ，即该函数返回本类（子类）内部的  `CRuntimeClass` 类型的静态公有成员变量 `classCDog` ，也就是链表头，其目的是为了方便后续遍历该链表

#### 4.2.2.3 `CObject::IsKindOf` 详解

以4.2.1节代码为例，我们使用了 `CObject` 的子类 `CDog` 调用 `IsKindOf` 函数：

```c++
dog.IsKindOf( RUNTIME_CLASS(CDog) );
```

**重要宏 `RUNTIME_CLASS` 回顾：**

首先，关注 `IsKindOf` 函数的参数，此代码中的参数为宏 `RUNTIME_CLASS` 的返回值，回顾4.2.2.2节可知，==**宏 `RUNTIME_CLASS(theClass)` 的返回值为其参数类内的 `CRuntimeClass` 类型的静态公有成员变量的地址，即 `(CRuntimeClass*)(&theClass::classtheClass)`**==，对应本例为： `((CRuntimeClass*)(&CDog::classCDog))` ，在下文中我们可以称其为链表头（原因见4.2.2.2），不过，这个链表头是相对的，不是整个链表中真正的链表头

我们将 `CRuntimeClass` 类型的静态变量从 `CDog` 开始向上展开，链表图如下所示：

![](.\assets\运行时类信息机制链表结构.png)

`CObject::IsKindOf` 的伪代码分析如下：

```c++
/*-------------------- 伪代码 --------------------*/

// 函数内部的this为&dog，函数实参为链表头，函数形参为：const CRuntimeClass* pClass
dog.IsKindOf( RUNTIME_CLASS(CDog) )
{
    // 利用&dog调用其内部宏展开的虚函数，获取链表头
    CRuntimeClass* pClassThis = GetRuntimeClass();
    
    // 该函数的返回值即为IsKindOf的返回值
    // 函数内部的this为链表头，函数实参也为链表头，函数形参为：const CRuntimeClass* pBaseClass
    return pClassThis->IsDerivedFrom(pClass)  // pClass == RUNTIME_CLASS(CDog)
    {
        const CRuntimeClass* pClassThis = this;  // 创建链表头副本
        while (pClassThis != NULL)
        {
            if (pClassThis == pBaseClass)  // pBaseClass == RUNTIME_CLASS(CDog)
				return true;
            else
                pClassThis = pClassThis->m_pBaseClass;
        }
        return false;
    }
}
```

`CObject::IsKindOf` 函数的原理就是==遍历「以调用该函数的对象的静态变量为链表头」的链表，依次比对这个链表头之后所有链表节点与函数参数实参是否相同==，具体过程如下：

- 利用对象（dog）的地址调用宏展开的虚函数 `GetRuntimeClass()` 获取本类静态变量的地址（链表头）
- 利用本类静态变量的地址（链表头）和目标进行比对
  - 如果相同，则证明对象属于这个类
  - 如果不同，则获取链表的下一个节点（其父类静态变量地址），以此类推，只要有一个相同即可证明对象属于这个类；若循环结束一次都没有比对成功，则证明对象不属于这个类

根据上述讲解，可以举一反三过一遍以下两次调用的过程：

```c++
dog.IsKindOf(RUNTIME_CLASS(CObject));
dog.IsKindOf(RUNTIME_CLASS(CWnd))
```



## 4.3 动态创建机制

> **「动态创建机制」**与「运行时类信息机制」的差别关键在于结构体 `CRuntimeClass` 的第四个成员 `CObject* (PASCAL* m_pfnCreateObject)();` 是否为空
>
> ==**「动态创建机制」的作用是：“在不事先知道类名的情况下”，将类的对象创建出来。**==

### 4.3.1 动态创建机制的使用

当一个类具备下列三个要素后，==`CRuntimeClass::CreateObject` （对象加工厂）函数就可以将类的对象创建出来==：

- 类必须派生自 `CObject`，直接派生或间接派生均可
- 类内必须添加声明宏 `DECLARE_DYNCREATE( theClass )` ，`theClass` 为本类类名
- 类外必须添加实现宏 `IMPLEMENT_DYNCREATE( theClass, baseClass )`，`baseClass` 为父类类名

示例代码如下：

- 定义一个 `CAnimal` 类，派生自 `CObject` 类，类内添加声明宏 `DECLARE_DYNAMIC( theClass )` 、类外添加实现宏 `IMPLEMENT_DYNCREATE( theClass, baseClass )` （运行时类信息机制）
- 定义一个 `CDog` 类，派生自 `CAnimal` 类，类内添加声明宏 `DECLARE_DYNCREATE( theClass )` 、类外添加实现宏 `IMPLEMENT_DYNCREATE( theClass, baseClass )` （动态创建机制）

```c++
#include <afxwin.h>
#include <iostream>

using namespace std;

class CAnimal : public CObject {
	DECLARE_DYNAMIC(CAnimal)
};
IMPLEMENT_DYNAMIC(CAnimal, CObject)

class CDog : public CAnimal {
	DECLARE_DYNCREATE( CDog )
};
IMPLEMENT_DYNCREATE( CDog, CAnimal )

int main() {
	CObject* pob = RUNTIME_CLASS(CDog)->CreateObject();  // 父类指针指向子类对象
	if (pob) {
		cout << pob << endl;  // 若成功创建CDog的对象，则打印其地址
	} else {
		cout << "对象创建失败" << endl;
	}
	return 0;
}
```

### 4.3.2 动态创建机制剖析

#### 4.3.2.1 宏展开详解

探究「动态创建机制」时，可以聚焦于「动态创建机制」与「运行时类信息机制」之间的差别

为深入理解动态创建机制，继4.3.1节代码，将上文所提到的声明宏 `DECLARE_DYNCREATE( theClass )` 与实现宏 `IMPLEMENT_DYNCREATE( theClass, baseClass )` 在 `CDog` 类中彻底展开，代码如下所示：

```c++
#include <afxwin.h>
#include <iostream>

using namespace std;

class CAnimal : public CObject{
	DECLARE_DYNAMIC( CAnimal )
};
IMPLEMENT_DYNAMIC( CAnimal, CObject )

class CDog : public CAnimal{
//	DECLARE_DYNCREATE( CDog ) 彻底展开为下文4行
//	DECLARE_DYNAMIC( CDog ) 将DECLARE_DYNCREATE展开后包含的DECLARE_DYNAMIC展开
public: 
	static const CRuntimeClass classCDog; 
	virtual CRuntimeClass* GetRuntimeClass() const; 
	static CObject* PASCAL CreateObject();  // DECLARE_DYNCREATE多出的函数声明
};

//IMPLEMENT_DYNCREATE( CDog, CAnimal ) 彻底展开为3个实现
//IMPLEMENT_DYNAMIC( CDog, CAnimal ) 将IMPLEMENT_DYNCREATE展开后包含的IMPLEMENT_DYNAMIC展开
AFX_COMDAT const CRuntimeClass CDog::classCDog = { 
		"CDog", 
		sizeof(class CDog), 
		0xFFFF, 
		CDog::CreateObject,	// 与运行时类信息机制不同，这里不再为空
		RUNTIME_CLASS(CAnimal), 
		NULL, 
		NULL 
}; 
CRuntimeClass* CDog::GetRuntimeClass() const 
{ 
	return RUNTIME_CLASS(CDog);
}
/*
 * IMPLEMENT_DYNCREATE多出的函数实现
 * 注：与main函数中调用的CreateObject()不是同一个函数，那个是CRuntimeClass的类内函数，但二者有关联
 */
CObject* PASCAL CDog::CreateObject()
{
	return new CDog;
}

int main(){
	CObject* pob = RUNTIME_CLASS(CDog)->CreateObject( );  // 父类指针指向子类对象
	if (pob) {
		cout << pob << endl;  // 若成功创建CDog的对象，则打印其地址
	} else {
		cout << "对象创建失败" << endl;
	}
	return 0;
}
```

展开后我们可以看到，「动态创建机制」与「运行时类信息机制」宏的区别在于：

- 增加了一个名为 `CreateObject()` 的静态函数，用于返回当前对象地址
- 在对 `CRuntimeClass` 类型的静态变量定义时第4个参数不再为 `NULL` ，而替代为新增加的名为 `CreateObject()` 的静态函数地址（名称）

我们将 `CRuntimeClass` 类型的静态变量从 `CDog` 开始向上展开，链表图如下所示：

![](.\assets\动态创建机制链表结构.png)

#### 4.3.2.2 `CRuntimeClass::CreateObject` 详解

`CRuntimeClass::CreateObject` 的伪代码分析如下：

```c++
/*-------------------- 伪代码 --------------------*/

RUNTIME_CLASS(CDog)->CreateObject( )  // 函数内部的this指针为本类（CDog）的静态变量地址（链表头）
{
    CObject* pObject = (*m_pfnCreateObject)()  // 静态变量第4个参数CDog::CreateObject
    {
        return new CDog;
    }
    return pObject;  // 返回CDog类对象地址
}
```

将 `CRuntimeClass::CreateObject` 调用过程总结如下：

1. 利用宏 `RUNTIME_CLASS( theClass )` 获取参数类 `theClass ` 的 `CreateObject` 类型的静态变量 `classtheClass` ，再通过该静态变量调用其内部名为 `CreateObject::CreateObject()` 的成员函数
2. 在 `CreateObject::CreateObject()` 中，获取静态变量第4个参数 `theclass::CreateObject` （ `m_pfnCreateObject` ），并调用之
3. 在 `theclass::CreateObject` 中，完成 `theClass` 类的对象的创建，并逐层返回
