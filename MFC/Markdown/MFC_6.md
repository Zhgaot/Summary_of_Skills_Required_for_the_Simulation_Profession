# 6 文档



## 6.1 文档类

### 6.1.1 基本概念

文档的相关类为 `CDocument` 类，它是一个用于管理数据的类，其封装了关于数据的管理（例如：数据提取、数据转换、数据存储等），并能和视图类 `CView` 及其子类进行交互（「文档类」在后台管理数据，「视图类」在前台显示数据）

### 6.1.2 文档类的基本使用

以下为添加文档类后最基本的代码示例；相较于上一章节的代码，我们添加了继承于 `CDocument` 的文档类 `CMyDoc` ，并在视图类 `CMyView` 中添加了支持动态创建机制的宏，代码如6-1所示：

```c++
/* ------------------------------ 代码 6-1 ------------------------------ */
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyDoc : public CDocument {};

class CMyView : public CView {
	DECLARE_DYNCREATE(CMyView)  // 动态创建机制
public:
	virtual void OnDraw(CDC* pDC);
};
IMPLEMENT_DYNCREATE(CMyView, CView)
void CMyView::OnDraw(CDC* pDC) {}

class CMyFrameWnd : public CFrameWnd {};

class CMyWinApp : public CWinApp {
public:
	virtual BOOL InitInstance();
};
BOOL CMyWinApp::InitInstance() {
	CMyFrameWnd* pFrame = new CMyFrameWnd;
	CMyDoc* pDoc = new CMyDoc;

	CCreateContext cct;
	cct.m_pCurrentDoc = pDoc;//文档类对象地址
	cct.m_pNewViewClass = RUNTIME_CLASS(CMyView);//&CMyView::classCMyView

	pFrame->LoadFrame(IDR_MENU1, WS_OVERLAPPEDWINDOW, NULL, &cct);
	this->m_pMainWnd = pFrame;
	pFrame->ShowWindow(SW_SHOW);
	pFrame->UpdateWindow();
	return TRUE;
}
CMyWinApp theApp;
```

基于以上代码，与之前代码最大不同在于：在函数 `CMyWinApp::InitInstance()` 中利用框架类对象创建框架窗口时，不再使用 `Create` 函数，而使用  `LoadFrame` 函数，这是添加文档类后的窗口创建之关键；添加文档类后的窗口创建过程如下：

1. 利用框架类对象地址 `pFrame` 调用  `LoadFrame` 函数，以创建框架窗口（在  `LoadFrame` 函数内部会调用 `Create` 函数）
2. 由于 `CMyFrameWnd` 没有重写 `OnCreate` 函数，则由 `CMyFrameWnd` 的父类 `CFrameWnd` 使用 `OnCreate` 函数处理框架窗口的 `WM_CREATE` 消息，在父类 `CFrameWnd` 处理框架窗口的 `WM_CREATE` 消息时，动态创建视图类对象，并创建视图窗口
3. 由于 `CMyView` 没有重写 `OnCreate` 函数，则由 `CMyView` 的父类 `CView` 使用 `OnCreate` 函数处理视图窗口的 `WM_CREATE` 消息，在父类 `CView` 处理视图窗口的 `WM_CREATE` 消息时，将文档类对象与视图类对象建立关联关系

### 6.1.3 程序创建过程

#### 6.1.3.1 `OnCreate` 函数参数

在学习Win32时了解到，处理 `WM_CREATE` 消息的函数包含两个参数：

1. `wParam`：不使用
2. `lParam`：形参为 `long` 型，实参为 `CREATESTRUCT` 类型的指针，该指针指向的位置保存了创建窗口函数 `::CreateWindowEx` 的全部12个参数

在MFC中，结合消息映射机制，如果添加了宏 `ON_WM_CREATE` ，则就会在窗口创建时抛出 `WM_CREATE` 消息，而MFC中处理 `WM_CREATE` 消息的函数为：

```c++
afx_msg int OnCreate( LPCREATESTRUCT pcs );
```

该函数的参数 `LPCREATESTRUCT pcs` ，其实就对应了Win32中处理 `WM_CREATE` 消息的函数参数2 `lParam` ，即 `pcs` 中保存了创建窗口函数 `::CreateWindowEx` 的全部12个参数

#### 6.1.3.2 程序创建过程

接6.1.2节，为在调试中更好地探寻其调用过程，在代码6-1的基础上，我们在 `CMyFrameWnd` 类和 `CMyView` 类中添加消息映射宏并重写父类的 `OnCreate` 函数，但在函数中还是调用父类的 `OnCreate` 函数，修改后的代码如6-2所示：

```c++
/* ------------------------------ 代码 6-2 ------------------------------ */
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyDoc : public CDocument {};

class CMyView : public CView {
	DECLARE_MESSAGE_MAP()
	DECLARE_DYNCREATE(CMyView)  // 动态创建机制
public:
	virtual void OnDraw(CDC* pDC);
	afx_msg int OnCreate(LPCREATESTRUCT pcs);
};
IMPLEMENT_DYNCREATE(CMyView, CView)
BEGIN_MESSAGE_MAP(CMyView, CView)
	ON_WM_CREATE()
END_MESSAGE_MAP()
void CMyView::OnDraw(CDC* pDC) {}
int CMyView::OnCreate(LPCREATESTRUCT pcs) {
	return CView::OnCreate(pcs);
}

class CMyFrameWnd : public CFrameWnd {
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs);
};
BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
END_MESSAGE_MAP()
int CMyFrameWnd::OnCreate(LPCREATESTRUCT pcs) {
	return CFrameWnd::OnCreate(pcs);
}

class CMyWinApp : public CWinApp {
public:
	virtual BOOL InitInstance();
};
BOOL CMyWinApp::InitInstance() {
	CMyFrameWnd* pFrame = new CMyFrameWnd;
	CMyDoc* pDoc = new CMyDoc;

	CCreateContext cct;
	cct.m_pCurrentDoc = pDoc;
	cct.m_pNewViewClass = RUNTIME_CLASS(CMyView);

	pFrame->LoadFrame(IDR_MENU1, WS_OVERLAPPEDWINDOW, NULL, &cct);
	this->m_pMainWnd = pFrame;
	pFrame->ShowWindow(SW_SHOW);
	pFrame->UpdateWindow();
	return TRUE;
}
CMyWinApp theApp;
```

对应6.1.2节所述的第1、2、3点，我们分别在代码6-2的第48、33、21行打断点运行

【验证第1点】从对应第1点的 `LoadFrame` 函数进入，伪代码如6-3所示：

```c++
/* ------------------------------ 代码 6-3 (伪代码) ------------------------------ */
CMyFrameWnd* pFrame = new CMyFrameWnd;
CMyDoc* pDoc = new CMyDoc;

CCreateContext cct;
cct.m_pCurrentDoc = pDoc;
cct.m_pNewViewClass = RUNTIME_CLASS(CMyView);

// 函数内部的this指针为pFrame
// 实参cct的形参名称为pContext
pFrame->LoadFrame(..., &cct) {
    
    // 函数内部的this指针为pFrame
    // pContext(cct)对应着形参名pContext
    Create(..., pContext) {
        
        // 函数内部的this指针为pFrame
        // pContext(cct)对应着形参名lpParam
        CreateEx(..., (LPVOID)pContext) {
            CREATESTRUCT cs;
			...
            cs.lpCreateParams = lpParam;  // 将cct赋给cs.lpCreateParams
            CreateWindowEx(..., cs.lpCreateParams) {
                // 在该函数内部就创建了主框架窗口，至此可验证了上文第1点
                // 主框架窗口创建后，会立刻抛出WM_CREATE消息，那么下一步即查看框架类的OnCreate
            }
        }
        
    }
    
}
```

【验证第2点】在主框架窗口创建后，会立刻抛出框架窗口的 `WM_CREATE` 消息，那么直接让程序运行到第2点对应的断点出，即查看框架类处理框架窗口 `WM_CREATE` 消息的函数： `CFrameWnd::OnCreate(pcs)` ，伪代码如6-4所示：

```c++
/* ------------------------------ 代码 6-4 (伪代码) ------------------------------ */
// 函数内部的this指针为pFrame
// 参数可以获取伪代码6-3第23行的CreateWindowEx的12个参数（包含cct）
CFrameWnd::OnCreate(pcs) {
    CCreateContext* pContext = (CCreateContext*)pcs->lpCreateParams;  // 获取cct地址
    
    // 函数内部的this指针为pFrame
    // pContext(cct)对应着形参名pContext
    OnCreateHelper(pcs, pContext) {
        OnCreateClient(pcs, pContext) {
            CreateView(pContext, AFX_IDW_PANE_FIRST) {  // 该函数用于创建视图窗口
                // 动态创建CView的对象并返回CView的地址给pView（所以CView必须添加动态创建宏）
                CWnd* pView = (CWnd*)pContext->m_pNewViewClass->CreateObject();
                pView->Create(..., pContext) {  // 该函数通过CView对象创建视图窗口
                    
                    // 函数内部的this指针为pView
       				// pContext(cct)对应着形参名lpParam
                    CreateEx(..., (LPVOID)pContext) {
                        CREATESTRUCT cs;
                        ...
                        cs.lpCreateParams = lpParam;  // 将cct赋给cs.lpCreateParams
                        CreateWindowEx(..., cs.lpCreateParams) {
                            // 在该函数内部就创建了视图窗口，至此可验证了上文第2点
                            // 视图窗口创建后，会立刻抛出视图窗口的WM_CREATE消息
                        }
                    }
                    
                }
            }
        }
    }
}
```

【验证第3点】在视图窗口创建后，会立刻抛出视图窗口的 `WM_CREATE` 消息，那么直接让程序运行到第3点对应的断点出，即查看视图类处理视图窗口 `WM_CREATE` 消息的函数： `CView::OnCreate(pcs)` ，伪代码如下：

```c++
/* ------------------------------ 代码 6-5 (伪代码) ------------------------------ */
// 函数内部的this指针为pView
// 参数可以获取伪代码6-4第22行的CreateWindowEx的12个参数（包含cct）
CView::OnCreate(pcs) {
    CCreateContext* pContext = (CCreateContext*)lpcs->lpCreateParams;  // 获取cct地址
    
    // 函数内部的this指针为pDoc
    // 让文档类对象地址调用AddView并传入视图类对象地址pView（以将二者关联！）
    // 传入的参数this表示视图类对象地址pView，在函数中的形参名为pView
    pContext->m_pCurrentDoc->AddView(this) {
        m_viewList.AddTail(pView);  // 将视图类对象地址pView添加进文档类对象里面的链表成员
        pView->m_pDocument = this;  // 将文档类对象地址pDoc赋给pView的m_pDocument成员中
        // 即，在该函数中，视图对象与文档对象相互保存了对方的地址在自己的某个成员中
        // 至此可验证上文第3点
    }
}
```

### 6.1.4 对象关系图

当代码中包含「应用程序类」、「框架类」、「视图类」、「文档类」后的对象关系图如下所示：

![](.\assets\添加文档类后的对象关系图.png)

由代码6-5可知：

- 文档类对象用一个链表成员变量 `m_viewList` ，保存多个视图类对象地址
- 视图类对象用一个普通成员变量 `m_pDocument` ，保存一个文档类对象地址

经分析可知，==一个文档类对象可以对应多个视图类对象（视图窗口），而一个视图类对象（视图窗口）只能对应一个文档类对象==

### 6.1.5 窗口切分

既然一个文档类对象可以对应多个视图类对象（视图窗口），这说明一个框架窗口可以包含多个视图窗口，我们将这种模式称为「窗口切分」

1. **窗口切分相关类**

   窗口切分的相关类为 `CSplitterWnd` ——不规则框架窗口类，其封装了关于不规则框架窗口的操作

   注：一个框架窗口只有一个用户区，称为规则框架窗口，即口字形框架窗口；一个框架窗口包含多个用户区，则称为不规则框架窗口，如日字形、田字形、倒日形框架窗口

2. **窗口切分的使用**

   重写 `CFrameWnd` 类的成员虚函数 `OnCreateClient` ：

   - 在虚函数中调用 `CSplitterWnd::CreateStatic` 函数来创建不规则框架窗口
   - 在虚函数中调用 `CSplitterWnd::CreateView` 函数来创建视图窗口

3. **窗口切分示例**

   我们在代码6-2的基础上，修改 `CFrameWnd` 类，重写其成员虚函数 `OnCreateClient` ，其余代码不变，如6-6所示：

   ```c++
   /* ---------------------------- 代码 6-6 (部分代码) ---------------------------- */
   class CMyFrameWnd : public CFrameWnd {
   	DECLARE_MESSAGE_MAP()
   public:
   	afx_msg int OnCreate(LPCREATESTRUCT pcs);
   	virtual BOOL OnCreateClient(LPCREATESTRUCT pcs, CCreateContext* pContext);
   public:
   	CSplitterWnd split;  // 不规则框架窗口（不能放在OnCreateClient中，否则生命周期太短）
   };
   BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
   	ON_WM_CREATE()
   END_MESSAGE_MAP()
   int CMyFrameWnd::OnCreate(LPCREATESTRUCT pcs) {
   	return CFrameWnd::OnCreate(pcs);
   }
   BOOL CMyFrameWnd::OnCreateClient( LPCREATESTRUCT pcs, CCreateContext* pContext ){
   	// 创建两个视图窗口
   	split.CreateStatic( this, 1, 2 );  // 一行两列
   	split.CreateView( 0, 0, RUNTIME_CLASS(CMyView), CSize(100,100), pContext ); 
   	split.CreateView( 0, 1, pContext->m_pNewViewClass, CSize(100,100), pContext );
   	return TRUE;
   }
   ```

   代码运行效果如下所示：

   ![](E:\航天三院\GitHub\Summary_of_Skills_Required_for_the_Simulation_Profession\MFC\Markdown\assets\窗口切分运行效果.png)

### 6.1.6 命令消息处理顺序

#### 6.1.6.1 活动视图窗口

1. **概念**

   根据6.1.4节的对象关系图可知，一个「框架类对象」中仅包含一个「视图类对象 `m_pViewActive` 」，但是一个「框架窗口」又可以包含多个「视图窗口」，这看似是矛盾的，但其实，顾名思义，「框架类对象」中的成员变量 `m_pViewActive` 中保存的是「活动视图窗口」

2. **如何设置**

   - 方法一（程序运行后设置）：

     如6.1.5节所示，当程序运行后，鼠标点击哪个窗口，哪个窗口就是「活动视图窗口」

   - 方法二（设置默认活动视图窗口）：

     接6.1.5节，因为视图类的对象不是我们创建的，而是交给了 MFC 创建，因此我们无法直接拿到视图类的对象并将其赋给 `m_pViewActive` 成员，因此正确设置默认视图窗口的方法为：==在函数 `CMyFrameWnd::OnCreateClient` 中当创建完多个视图窗口后，使用函数 `split.GetPane(行, 列)` 来获取某个视图窗口的对象，并将其强转为 `CView*` 类型后，赋值给 `m_pViewActive` 成员；==我们在代码6-6的基础上更改（仅展示函数 `CMyFrameWnd::OnCreateClient` 部分），如代码6-7所示：

     ```c++
     /* --------------------------- 代码 6-7 (部分代码) --------------------------- */
     BOOL CMyFrameWnd::OnCreateClient(LPCREATESTRUCT pcs, CCreateContext* pContext)
     {
     	// 创建两个视图窗口
     	split.CreateStatic(this, 1, 2);  // 一行两列
     	split.CreateView(0, 0, RUNTIME_CLASS(CMyView), CSize(100, 100), pContext); 
     	split.CreateView(0, 1, RUNTIME_CLASS(CMyView), CSize(100, 100), pContext);
         
         // 设置默认活动视图窗口
     	m_pViewActive = (CView*)split.GetPane(0, 0);  // 第0行0列的视图窗口对象
     	return TRUE;
     }
     ```

如果不按照 “方法二” 在程序运行前设置默认活动视图窗口，那么程序运行后 `m_pViewActive` 指向为空，表明没有活动视图窗口，这将导致其他问题；例如：在创建菜单项后，如果 `m_pViewActive` 并未指向任何一个活动视图窗口，即使处理了菜单项对应的 `ON_COMMAND` 消息，菜单项依然是无法点击的灰色状态，此时只能按照 “方法一” 来手动设置活动视图窗口才能点击菜单项

#### 6.1.6.2 命令消息处理顺序

继5.4节，本节验证了在加入「文档类」后，命令消息（ `ON_COMMAND` ）的处理顺序是如何；如代码6-8所示，我们在「视图类」、「文档类」、「框架类」、「应用程序类」中均添加了消息映射宏，并处理了 `ON_COMMAND` 消息，在依次进行了 “不屏蔽任何代码” 、“屏蔽29行（视图类处理 `ON_COMMAND` 消息）” 、“屏蔽12行（文档类处理 `ON_COMMAND` 消息）” 、“屏蔽48行（框架类处理 `ON_COMMAND` 消息）” 的操作后，发现==程序选择处理命令消息 `ON_COMMAND` 的顺序为：「视图类」>「文档类」>「框架类」>「应用程序类」==：

```c++
/* ---------------------------------- 代码 6-8 ---------------------------------- */
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyDoc : public CDocument {
	DECLARE_MESSAGE_MAP()
public:
	afx_msg void OnNew();
};
BEGIN_MESSAGE_MAP(CMyDoc, CDocument)
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
void CMyDoc::OnNew() {
	AfxMessageBox("文档类处理的WM_COMMAND消息");
}

class CMyView : public CView {
	DECLARE_MESSAGE_MAP()
	DECLARE_DYNCREATE(CMyView)  // 动态创建机制
public:
	virtual void OnDraw(CDC* pDC);
	afx_msg int OnCreate(LPCREATESTRUCT pcs);
	afx_msg void OnNew();
};
IMPLEMENT_DYNCREATE(CMyView, CView)
BEGIN_MESSAGE_MAP(CMyView, CView)
	ON_WM_CREATE()
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
void CMyView::OnDraw(CDC* pDC) {}
int CMyView::OnCreate(LPCREATESTRUCT pcs) {
	return CView::OnCreate(pcs);
}
void CMyView::OnNew() {
	AfxMessageBox("视图类处理的WM_COMMAND消息");
}

class CMyFrameWnd : public CFrameWnd {
	DECLARE_MESSAGE_MAP()
public:
	afx_msg void OnNew();
	virtual BOOL OnCreateClient(LPCREATESTRUCT pcs, CCreateContext* pContext);
public:
	CSplitterWnd split;  // 不规则框架窗口（不能将其放在OnCreateClient函数中，否则生命周期太短）
};
BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
void CMyFrameWnd::OnNew() {
	AfxMessageBox("框架类处理的WM_COMMAND消息");
}
BOOL CMyFrameWnd::OnCreateClient(LPCREATESTRUCT pcs, CCreateContext* pContext) {
	// 创建两个视图窗口
	split.CreateStatic(this, 1, 2);  // 一行两列
	split.CreateView(0, 0, RUNTIME_CLASS(CMyView), CSize(100, 100), pContext);  // 倒数第二个参数无用
	split.CreateView(0, 1, pContext->m_pNewViewClass, CSize(100, 100), pContext);
	m_pViewActive = (CView*)split.GetPane(0, 0);
	return TRUE;
}

class CMyWinApp : public CWinApp {
	DECLARE_MESSAGE_MAP()
public:
	afx_msg void OnNew();
	virtual BOOL InitInstance();
};
BEGIN_MESSAGE_MAP(CMyWinApp, CWinApp)
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
void CMyWinApp::OnNew() {
	AfxMessageBox("应用程序类处理的WM_COMMAND消息");
}
BOOL CMyWinApp::InitInstance() {
	CMyFrameWnd* pFrame = new CMyFrameWnd;
	CMyDoc* pDoc = new CMyDoc;

	CCreateContext cct;
	cct.m_pCurrentDoc = pDoc;  // 文档类对象地址
	cct.m_pNewViewClass = RUNTIME_CLASS(CMyView);  // &CMyView::classCMyView

	pFrame->LoadFrame(IDR_MENU1, WS_OVERLAPPEDWINDOW, NULL, &cct);
	this->m_pMainWnd = pFrame;
	pFrame->ShowWindow(SW_SHOW);
	pFrame->UpdateWindow();
	return TRUE;
}
CMyWinApp theApp;
```

#### 6.1.6.3 命令消息处理顺序原理（略）

### 6.1.7 文档类和视图类的交互

本节将简单讲解几个文档类和视图类中常用的用于在二者交互时使用的成员函数

#### 6.1.7.1 文档类成员函数

- **`void CDocument::UpdateAllViews(CView* pSender, LPARAM lHint = 0L, CObject* pHint = NULL);`**
  - **功能：**当文档类数据发生变化时，调用该函数可刷新和文档类对象相关联的视图类对象（即为视图类对象产生绘图消息，让视图类对象调用一次 `OnDraw` 函数，那么可以在视图类的 `OnDraw` 函数中编写当文档类数据发生变化后视图类对象如何将数据显示在视图窗口上）
  - **传参：**主要针对第一个参数，如果传入 `NULL` ，即为所有与文档类对象相关联的视图类对象产生绘图消息；如果传入 `CView*` 类型的变量（某视图类对象地址），即为==除了该视图类对象以外的==所有与文档类对象相关联的视图类对象产生绘图消息

- **`POSITION CDocument::GetFirstViewPosition() const;`**
  - **功能：**查看6.1.4所示的对象关系图可知，文档类里的 `m_viewList` 链表成员包含了所有与该文档类对象相关联的视图类对象，但微软并未提供直接的接口来操作该链表；==通过该函数，可以获得 `m_viewList` 链表的头节点的前一个节点的迭代器==
  - **返回值：**返回值类型为 `POSITION`

- **`CView* CDocument::GetNextView(POSITION& rPosition) const;`**
  - **功能：**将 `POSITION` 类型的迭代器向后移动一次，并得到其对应的 `CView*` 类型的视图窗口对象地址
  - **传参：**传入 `POSITION` 类型的迭代器，例如传入 `GetFirstViewPosition` 函数获得的迭代器
  - **返回值：**移动后的 `POSITION` 类型的迭代器对应的 `CView*` 类型的视图窗口对象地址


注：对于MFC中所有链表，微软均不会提供直接的接口，但是都可以通过 `GetFirstXXXPosition` 来获得链表头的前一个节点的迭代器、通多 `GetNextXXX` 移动链表迭代器并获得该迭代器对应的数据域内容

#### 6.1.7.2 视图类成员函数

- **`CDocument* CView::GetDocument()`**
  - **功能：**获取视图类对象相关联的文档类对象
  - **返回值：** `CDocument*` 类型的变量，但可以强转为自定义的 `CDocument` 子类的类型

#### 6.1.7.3 代码示例

```c++
/* ---------------------------------- 代码 6-9 ---------------------------------- */
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyDoc : public CDocument {
	DECLARE_MESSAGE_MAP()
public:
	afx_msg void OnNew();
	CString m_str;  // 模拟文档类管理的数据
};
BEGIN_MESSAGE_MAP(CMyDoc, CDocument)
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
void CMyDoc::OnNew() {
	// 假定：点击新建菜单项后，m_str就会接收到数据
	// 需要将m_str中的数据显示在右侧的视图窗口上
	this->m_str = "菜单项被点击，将该信息显示在右侧视图窗口上";
	POSITION pos = this->GetFirstViewPosition();
	CView* pLeftView = this->GetNextView(pos);
	this->UpdateAllViews(pLeftView);
}

class CMyView : public CView {
	DECLARE_DYNCREATE(CMyView)  // 动态创建机制
public:
	virtual void OnDraw(CDC* pDC);
};
IMPLEMENT_DYNCREATE(CMyView, CView)
void CMyView::OnDraw(CDC* pDC) {
	//CMyDoc* pDoc = (CMyDoc*)this->m_pDocument;
	CMyDoc* pDoc = (CMyDoc*)this->GetDocument();
	pDC->TextOut(100, 100, pDoc->m_str);
}

class CMyFrameWnd : public CFrameWnd {
public:
	virtual BOOL OnCreateClient(LPCREATESTRUCT pcs, CCreateContext* pContext);
public:
	CSplitterWnd split;  // 不规则框架窗口（不能将其放在OnCreateClient函数中，否则生命周期太短）
};
BOOL CMyFrameWnd::OnCreateClient(LPCREATESTRUCT pcs, CCreateContext* pContext) {
	// 创建两个视图窗口
	split.CreateStatic(this, 1, 2);  // 一行两列
	split.CreateView(0, 0, RUNTIME_CLASS(CMyView), CSize(100, 100), pContext);  // 倒数第二个参数无用
	split.CreateView(0, 1, pContext->m_pNewViewClass, CSize(100, 100), pContext);
	m_pViewActive = (CView*)split.GetPane(0, 0);
	return TRUE;
}

class CMyWinApp : public CWinApp {
public:
	virtual BOOL InitInstance();
};
BOOL CMyWinApp::InitInstance() {
	CMyFrameWnd* pFrame = new CMyFrameWnd;
	CMyDoc* pDoc = new CMyDoc;

	CCreateContext cct;
	cct.m_pCurrentDoc = pDoc;  // 文档类对象地址
	cct.m_pNewViewClass = RUNTIME_CLASS(CMyView);  // &CMyView::classCMyView

	pFrame->LoadFrame(IDR_MENU1, WS_OVERLAPPEDWINDOW, NULL, &cct);
	this->m_pMainWnd = pFrame;
	pFrame->ShowWindow(SW_SHOW);
	pFrame->UpdateWindow();
	return TRUE;
}
CMyWinApp theApp;
```



## 6.2 单文档视图架构

> 单文档视图架构指的是：**只能管理一个文档，即只有一个文档类对象**的架构

### 6.2.1 基本代码框架

#### 6.2.1.1 涉及的类

- 参与架构的类：`CWinApp` / `CFrameWnd` / `CView` / `CDocument`
- 需要使用的类：
  - `CDocTemplate`（文档模板类）的子类 `CSingleDocTemplate`（单文档模板类）
  - `CDocManager`（文档管理类）

#### 6.2.1.2 代码示例

```c++
/* ---------------------------------- 代码 6-10 ---------------------------------- */
#include <afxwin.h>
#include "resource.h"
class CMyDoc : public CDocument{
	DECLARE_DYNCREATE( CMyDoc )
};
IMPLEMENT_DYNCREATE( CMyDoc, CDocument )

class CMyView : public CView{
	DECLARE_DYNCREATE( CMyView )
public:
	virtual void OnDraw( CDC* pDC );
};
IMPLEMENT_DYNCREATE( CMyView, CView )
void CMyView::OnDraw( CDC* pDC ){
	pDC->TextOut( 100, 100, "我是视图窗口" );
}

class CMyFrameWnd : public CFrameWnd{
	DECLARE_DYNCREATE( CMyFrameWnd )
};
IMPLEMENT_DYNCREATE( CMyFrameWnd, CFrameWnd )

class CMyWinApp : public CWinApp{
public:
	virtual BOOL InitInstance( );
};
BOOL CMyWinApp::InitInstance( ){
	CSingleDocTemplate* pTemplate = new CSingleDocTemplate(
        									IDR_MENU1,
        									RUNTIME_CLASS(CMyDoc),
        									RUNTIME_CLASS(CMyFrameWnd),
											RUNTIME_CLASS(CMyView)
    									);
	AddDocTemplate( pTemplate );
	OnFileNew( );
	m_pMainWnd->ShowWindow( SW_SHOW );
	m_pMainWnd->UpdateWindow( );
	return TRUE;
}
CMyWinApp theApp;
```

手写单文档视图架构如代码6-10所示，代码要点如下：

- 在参与架构的4个中，除应用程序类 `CMyWinApp` 外，其余的3个类均支持动态创建机制，这样写是希望：应用程序类 `CMyWinApp` 的对象由程序员自己创建，而其他3个类的对象由MFC来动态创建

  - 不要忘记为视图类重写虚函数 `OnDraw` ，否则MFC也无法创建视图类的对象

- 应用程序类的入口函数 `CMyWinApp::InitInstance` 有变化，函数内部各语句执行过程参照6.2.2节

  - 函数 `CSingleDocTemplate` 的第一个参数需传入一个资源ID，这里填入菜单资源 `IDR_MENU1`

- 仅编写6-10代码并添加菜单资源代码依然无法运行，还缺少MFC所需要的一个字符串资源 `AFX_IDS_UNTITLED` ，其对应单文档视图架构的标题名，因此需添加该字符串资源：

  ![单文档视图架构添加字符串资源](.\assets\单文档视图架构添加字符串资源.png)

代码的运行结果如下所示：

![](.\assets\手写单文档视图架构运行结果.png)

### 6.2.2 执行过程详述





















