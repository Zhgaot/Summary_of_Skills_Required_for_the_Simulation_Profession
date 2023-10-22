# 5 视图窗口

## 5.1 基本概念

视图窗口**显示：**

「视图窗口」是覆盖在「主框架窗口」用户区上的另一个窗口，该窗口没有标题栏、没有边框、背景为白色，并将「主框架窗口」的用户区完全遮挡上

视图窗口**功能：**

「视图窗口」是一个专门用于显示数据和画图的的窗口；虽然「主框架窗口」的用户区也可以实现相同功能，但在 MFC 中，每个类专门负责一类功能，使其模块化，例如，「主框架窗口」专门的功能为「容器」

视图窗口**相关类：**

**`CView` 及其子类**，其父类为 `CWnd` 类（所有窗口类的最基类），它封装了关于「视图窗口」的各种操作，以及和「文档类」的数据交互（「文档类」在后台管理数据，「视图类」在前台显示数据）



## 5.2 视图窗口的使用

### 5.2.1 使用要点

视图窗口的使用要点：

1. 定义一个自己的视图类（ `CMyView` ），派生自 `CView` ，并重写父类成员纯虚函数 `OnDraw()` 

2. 其余的框架类和应用程序类代码不变

3. 在处理「框架窗口」的 `WM_CREATE` 消息时，定义 `CMyView` 类对象，并调用 `Create` 函数创建视图窗口； `Create` 函数的最后一个参数视图窗口ID填写的值建议大于 `AFX_IDW_PANE_FIRST` 

使用要点说明：

- 对于第1点，其原因将在后文阐述
- 对于第3点：
  - 「视图窗口」是「主框架窗口」的子窗口，那么根据WIN32的知识可知，子窗口是在父窗口处理 `WM_CREATE` 消息时所创建的
  - 当 `Create` 函数的「视图窗口ID」这一参数填写的值大于 `AFX_IDW_PANE_FIRST` 时（该宏定义对应的数值为 `0xE900` ）， `Create` 函数会修改「视图窗口」的大小使其适应「框架窗口」客户区的大小，那么这也将导致 `Create` 函数的第4个参数（用于设置视图窗口的位置与范围）无效

### 5.2.2 示例代码

基本的视图窗口示例代码如下：

```c++
#include <afxwin.h>
#include "resource.h"

class CMyView : public CView {
public:
	void OnDraw(CDC* pDC);  // 父类的纯虚函数，必须重写，否则无法创建CView的对象
};

void CMyView::OnDraw(CDC* pDC) {
	pDC->TextOut(100, 100, "CMyView::OnDraw");
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
	CMyView* pView = new CMyView;				// 在堆区创建视图类对象，否则生命周期太短了
	pView->Create(
		NULL,									// 类名称
		"MFCView",								// 标题栏信息（在视图窗口处无效）
		WS_CHILD | WS_VISIBLE | WS_BORDER,		// 子窗口风格与边框风格
		CRect(0, 0, 200, 200),					// 视图窗口位置与范围
		this,									// 父窗口（CMyFrameWnd）
		AFX_IDW_PANE_FIRST						// 视图窗口ID（框架窗口中会有多个视图窗口）
	);
	return CFrameWnd::OnCreate(pcs);
}

class CMyWinApp : public CWinApp {
public:
	virtual BOOL InitInstance();
};

BOOL CMyWinApp::InitInstance() {
	CMyFrameWnd* pFrame = new CMyFrameWnd;
	pFrame->Create( NULL, "MFCView");
	this->m_pMainWnd = pFrame;
	pFrame->ShowWindow(SW_SHOW);
	pFrame->UpdateWindow();
	return TRUE;
}

CMyWinApp theApp;
```

程序运行结果如下所示：

![](.\assets\视图窗口最基础示例代码显示.png)



## 5.3 纯虚函数 `CView::OnDraw()` 讲解

在5.2.2示例代码中的第9~11行重写的父类纯虚函数 `OnDraw` 和程序运行结果界面上显示的字符串可知， `OnDraw` 函数可以用来绘图；但是，通常我们使用的是绘图消息 `ON_WM_PAINT()` 来绘图，那么 `OnDraw` 函数与绘图消息 `ON_WM_PAINT()` 的区别与联系即为本节重点

### 5.3.1 引例

如果我们在 `CMyView` 类中既使用 `OnPaint()` 函数处理绘图消息 `ON_WM_PAINT()` ，又重写父类纯虚函数 `OnDraw` ，代码如下所示（仅展示 `CMyView` 类的相关代码）：

```c++
class CMyView : public CView {
	DECLARE_MESSAGE_MAP()
public:
	void OnDraw(CDC* pDC);  // 父类的纯虚函数，必须重写，否则无法创建CView的对象
	afx_msg void OnPaint();  // 绘图消息处理函数
};
BEGIN_MESSAGE_MAP( CMyView, CView )
	ON_WM_PAINT()  // 绘图消息
END_MESSAGE_MAP()

void CMyView::OnDraw(CDC* pDC) {
	pDC->TextOut(100, 100, "CMyView::OnDraw");
}

void CMyView::OnPaint( ){
	PAINTSTRUCT ps = { 0 };
	HDC hdc = ::BeginPaint( this->m_hWnd, &ps );
	::TextOut( hdc, 200, 200, "CMyView::OnPaint", strlen("CMyView::OnPaint") );
	::EndPaint( this->m_hWnd, &ps );
}
```

程序执行结果如下所示：

![](.\assets\视图窗口自己处理绘图消息.png)

与5.2.2节的程序执行过程对比可知：==当既使用 `OnPaint()` 函数处理绘图消息 `ON_WM_PAINT()` ，又重写父类纯虚函数 `OnDraw` 时，程序会优先执行前者==

### 5.3.2 原理

根据第3章讲解的消息映射机制：

- 如果在 `CMyView` 类中使用 `OnPaint()` 函数处理了绘图消息 `ON_WM_PAINT()` ，那么在函数 `AfxWndProc` 内的函数 `AfxCallWndProc` 遍历链表时，会首先遍历到 `CMyView` 类中对于绘图消息的处理函数 

- 如果在 `CMyView` 类中仅重写父类纯虚函数 `OnDraw` ，那么在函数 `AfxWndProc` 内的函数 `AfxCallWndProc` 遍历链表时，会遍历到 `CMyView` 类的父类 `CView` 中对于绘图消息的处理函数 `CView::OnPaint()` ，而 `CView::OnPaint()` 函数内部会调用到子类重写的 `CMyView::OnDraw()` 函数：

  ![](.\assets\CMyView的OnDraw调用堆栈.png)

本节所阐述的消息映射机制的链表示意如下图：

![](.\assets\视图类_消息映射机制原理示意.png)



## 5.4 命令消息处理顺序

### 5.4.1 引言

对于「标准Windows消息宏 ( `ON_WM_XXX` ) 」和「自定义消息宏 ( `ON_MESSAGE` ) 」，一条消息是哪个窗口的消息，就找到该消息对应的类（或者该类的支脉）上的消息处理函数；但是，==对于「命令消息宏 ( `ON_COMMAND` ) 」，这种消息所有类都可以处理它==；另外，「应用程序类」因为没有对应的窗口，所以不会处理任何其他类型的消息，而只能够处理 `ON_COMMAND` 消息

本节将探讨在仅有「应用程序类」、「框架类」、「视图类」的情况下，当它们均定义了处理 `ON_COMMAND` 消息的消息处理函数时，程序会优先选择哪个类来处理 `ON_COMMAND` 消息

### 5.4.2 代码示例

首先，我们创建一个与3.5节相同的菜单资源，即包括：主菜单/顶部菜单（ID为 `IDR_MENU1` ，名称为“文件”）和子菜单/下拉式菜单（ID为 `ID_NEW` ，名称为“新建”），创建过程不予赘述；创建菜单资源后，在创建「主框架窗口」的函数 `CMyWinApp::InitInstance()` 中调用 `Create()` 函数时将菜单挂载：

```c++
BOOL CMyWinApp::InitInstance() {
	CMyFrameWnd* pFrame = new CMyFrameWnd;
	pFrame->Create(NULL, "MFCView", WS_OVERLAPPEDWINDOW, CFrameWnd::rectDefault,
		NULL, (CHAR*)IDR_MENU1);
	this->m_pMainWnd = pFrame;
	pFrame->ShowWindow(SW_SHOW);
	pFrame->UpdateWindow();
	return TRUE;
}
```

而后，分别试验如下情况：

1. 在「应用程序类」、「框架类」、「视图类」中均处理 `ON_COMMAND` 消息，具体处理的消息为：点击子菜单/下拉式菜单（ID为 `ID_NEW` ，名称为“新建”），结果为程序调用了「视图类」对于 `ON_COMMAND` 消息的消息处理函数

2. 仅在「应用程序类」、「框架类」中均处理 `ON_COMMAND` 消息，结果为程序调用了「框架类」对于 `ON_COMMAND` 消息的消息处理函数

3. 仅在「应用程序类」中均处理 `ON_COMMAND` 消息，结果为程序调用了「应用程序类」对于 `ON_COMMAND` 消息的消息处理函数

需要注意的是：对于试验1，即如果在「视图类」中均处理了菜单的 `ON_COMMAND` 消息，必须点击一下视图窗口，这样可以将被点击的视图窗口设置为「活动窗口」（视图窗口可能不止一个，但是活动窗口仅有一个），此时才能够点击子菜单中的菜单项，否则子菜单中的菜单项是灰色不可点击的；或者，可以==在创建「视图窗口」后，将我们自己创建的视图对象赋给框架类的 `m_pViewActive` 成员，默认这个视图对象对应的视图窗口为活动窗口==，这样就无需在程序运行起来后，还使用鼠标点击视图窗口才能点击子菜单的菜单项了，代码如下所示：

```c++
int CMyFrameWnd::OnCreate(LPCREATESTRUCT pcs) {
	CMyView* pView = new CMyView;
	pView->Create( NULL, "MFCView", WS_CHILD | WS_VISIBLE | WS_BORDER,
                  CRect(0, 0, 200, 200), this, AFX_IDW_PANE_FIRST );
	this->m_pViewActive = pView;  // 设置默认活动窗口
	return CFrameWnd::OnCreate(pcs);
}
```

下面为试验1的全部代码，2和3仅需要注释掉「应用程序类」、「框架类」中实现宏中间的 `ON_COMMAND(ID_NEW, OnNew)` 即可：

```c++
#include <afxwin.h>
#include "resource.h"  // 菜单的ID（IDR_MENU1）需要该头文件支持

class CMyView : public CView {
	DECLARE_MESSAGE_MAP()
public:
	void OnDraw(CDC* pDC);  // 父类的纯虚函数，必须重写，否则无法创建CView的对象
	afx_msg void OnNew();
};
BEGIN_MESSAGE_MAP( CMyView, CView )
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()

void CMyView::OnDraw(CDC* pDC) {
	pDC->TextOut(100, 100, "CMyView::OnDraw");
}

void CMyView::OnNew() {
	AfxMessageBox("视图类处理了WM_COMMAND消息");
}

class CMyFrameWnd : public CFrameWnd {
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs);
	afx_msg void OnNew();
};
BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()

void CMyFrameWnd::OnNew() {
	AfxMessageBox("框架类处理了WM_COMMAND消息");
}

int CMyFrameWnd::OnCreate(LPCREATESTRUCT pcs) {
	CMyView* pView = new CMyView;
	pView->Create( NULL, "MFCView", WS_CHILD | WS_VISIBLE | WS_BORDER,
                  CRect(0, 0, 200, 200), this, AFX_IDW_PANE_FIRST );
	this->m_pViewActive = pView;  // 设置默认活动窗口
	return CFrameWnd::OnCreate(pcs);
}

class CMyWinApp : public CWinApp {
	DECLARE_MESSAGE_MAP()
public:
	virtual BOOL InitInstance();
	afx_msg void OnNew();
};
BEGIN_MESSAGE_MAP(CMyWinApp, CWinApp)
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()

void CMyWinApp::OnNew() {
	AfxMessageBox("应用程序类类处理了WM_COMMAND消息");
}

BOOL CMyWinApp::InitInstance() {
	CMyFrameWnd* pFrame = new CMyFrameWnd;
	pFrame->Create(NULL, "MFCView", WS_OVERLAPPEDWINDOW, CFrameWnd::rectDefault,
		NULL, (CHAR*)IDR_MENU1);
	this->m_pMainWnd = pFrame;
	pFrame->ShowWindow(SW_SHOW);
	pFrame->UpdateWindow();
	return TRUE;
}

CMyWinApp theApp;
```

总结：==在仅有「应用程序类」、「框架类」、「视图类」的情况下，当它们均定义了处理 `ON_COMMAND` 消息的消息处理函数时，程序选择处理 `ON_COMMAND` 消息的顺序为：「视图类」>「框架类」>「应用程序类」==

## 5.5 对象关系图

截至当前，学习过了「应用程序类」、「框架类」、「视图类」，从5.4.2节代码也可以看出：

- 「应用程序类」的对象为全局变量 `theApp`（ `CMyWinApp` ）
- 「框架类」的对象为堆区变量 `pFrame`（ `CMyFrameWnd*` ）
- 「应用程序类」的对象为堆区变量 `pView`（ `CMyView*` ）

它们三者的关系为：

![](.\assets\三个类对象之间的关系.png)

这表明，==如果想找到框架类对象或者视图类对象，只需要从 `theApp` 中获取即可，而 `theApp` 又是全局变量，取用较为方便==

