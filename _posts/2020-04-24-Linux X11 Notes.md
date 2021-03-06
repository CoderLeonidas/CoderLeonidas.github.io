---
layout:     post
title:      "Linux X11 Notes"
date:       2020-04-24 14:30:02
author:     "CoderLeonidas"
catalog: true
tags:

- Linux
- C++
- X11

---


# Linux X11 Notes

## X11

在X Window系统（X11，或简称X）是一个窗口系统为位图显示器，常见于类Unix 操作系统。

X提供了GUI环境的基本框架：在显示设备上绘制和移动窗口以及与鼠标和键盘进行交互。X并不要求用户界面–这是由单独的程序处理的。因此，基于X的环境的视觉样式差异很大。不同的程序可能呈现完全不同的界面。

![](https://i.loli.net/2020/04/24/hbpjxlv8zWCPQSu.png)

X使用客户端-服务器模型：X服务器与各种客户端程序进行通信。服务器接受图形输出的请求（窗口），并发送回用户输入（从键盘，鼠标或触摸屏）。该服务器可以用作：

* 一种在另一个显示系统的窗口中显示的应用程序
* 一种控制PC视频输出的系统程序
* 一种专用硬件

这种客户端-服务器术语-用户的终端是服务器，应用程序是客户端-通常会使新的X用户感到困惑，因为这些术语看起来是相反的。 但是X只是从应用程序的角度出发，而不是从最终用户的角度：X为应用程序提供显示和I/O服务，因此它是服务器； 应用程序使用这些服务，因此它们是客户端。

服务器和客户端之间的通信协议透明地运行在网络上:客户端和服务器可能运行在同一台机器上，也可能运行在不同的机器上，可能使用不同的体系结构和操作系统。客户端和服务器甚至可以通过在加密的网络会话上通过隧道连接在Internet上进行安全通信。

X客户机本身可以通过向其他客户机提供显示服务来模拟X服务器。这就是所谓的“X嵌套”。**Xnest**和**Xephyr**等开源客户端支持这种X嵌套。


要在远程机器上使用X客户机应用程序，用户可以执行以下操作

* 在本地机器上，打开终端窗口
* 使用带有X forwarding参数的ssh连接到远程机器
* 请求本地显示/输入服务（例如，export DISPLAY = [用户的计算机]：如果不使用启用了X转发的SSH，则为0）

然后，远程X客户机应用程序将连接到用户的本地X服务器，为用户提供显示和输入。

或者，本地计算机可以运行一个小程序，该程序连接到远程计算机并启动客户机应用程序。


远程客户端的实际例子包括：

* 以图形方式管理远程计算机(类似于使用远程桌面，但只有一个窗口)
* 使用客户端应用程序与协作工作组中的大量其他终端用户连接
* 在远程计算机上运行计算密集型模拟，并在本地桌面计算机上显示结果
* 在多台计算机上同时运行图形软件，由一个显示器、键盘和鼠标控制

## Macro

### 

```c++
unsigned long AllPlanes;

unsigned long BlackPixel(Display *display, int screen_number);
unsigned long WhitePixel(Display *display, int screen_number);
int ConnectionNumber(Display *display);
Colormap DefaultColormap(Display *display, int screen_number);
int DefaultDepth(Display *display, int screen_number);
int *XListDepths(Display *display, int screen_number, int count_return);
GC DefaultGC(Display *display, int screen_number);
Window DefaultRootWindow(Display *display);
Screen *DefaultScreenOfDisplay(Display *display);
int DefaultScreen(Display *display);
Visual *DefaultVisual(Display *display, int screen_number);
int DisplayCells(Display *display, int screen_number);
int DisplayPlanes(Display *display, int screen_number);
char *DisplayString(Display *display);
long XMaxRequestSize(Display *display)
long XExtendedMaxRequestSize(Display *display)
unsigned long LastKnownRequestProcessed(Display *display);
unsigned long NextRequest(Display *display);
int ProtocolVersion(Display *display);
int ProtocolRevision(Display *display);
int QLength(Display *display);
Window RootWindow(Display *display, int screen_number);
int ScreenCount(Display *display);
Screen *ScreenOfDisplay(Display *display, int screen_number);
char *ServerVendor(Display *display)
int VendorRelease(Display *display)
```


AllPlanes宏返回一个值，该值的所有位都设置为1，适合在一个过程的平面参数中使用。

BlackPixel宏返回指定屏幕的黑色像素值。

DefaultScreen宏返回XOpenDisplay例程中引用的默认屏幕号。


## XStandardColormap

使用调色板、平滑阴影绘图或数字化图像的应用程序需要大量的颜色。此外，这些应用程序通常需要从颜色三元组到显示适当颜色的像素值的有效映射。

例如，考虑一个三维显示程序，它想要绘制一个平滑的阴影球体。在球体图像的每个像素处，程序计算反射回观察者的光的强度和颜色。每次计算的结果是范围为0.0到1.0的RGB系数的三倍。为了绘制球体，程序需要一个提供大范围均匀分布颜色的颜色映射。应该安排colormap，以便程序能够非常快速地将其RGB三元组转换为像素值，因为绘制整个球体需要许多这样的转换。

再举一个例子，假设一个用户正在运行一个图像处理程序来显示地球资源数据。图像处理程序需要一个颜色映射，设置为8种红色、8种绿色和4种蓝色，总共256种颜色。因为默认的colormap中已经使用了一些颜色，所以图像处理程序分配并安装一个新的colormap。

用户决定通过调用调色板程序来混合和选择颜色，从而改变图像中的一些颜色。调色板程序还需要一个包含8个红色、8个绿色和4个蓝色的colormap，因此就像图像处理程序一样，它必须分配和安装一个新的colormap。

```c++

typedef struct {
	Colormap colormap;
	unsigned long red_max;
	unsigned long red_mult;
	unsigned long green_max;
	unsigned long green_mult;
	unsigned long blue_max;
	unsigned long blue_mult;
	unsigned long base_pixel;
	VisualID visualid;
	XID killid;
} XStandardColormap;
```


## Event

### Event Types

事件是由X服务器异步生成的数据，它是某些设备活动的结果，或者是Xlib函数发送的请求的副作用。与设备相关的事件从源窗口传播到祖先窗口，直到某个客户端应用程序选择了该事件类型，或者直到显式地丢弃该事件。

X服务器通常只在客户端明确要求被告知该事件类型时才向客户端应用程序发送事件，通常通过设置窗口的event-mask属性来实现。当您创建一个窗口或通过更改窗口的事件掩码时，也可以设置掩码。您还可以通过操作窗口属性的do-not-propagate掩码来屏蔽将传播到祖先窗口的事件。然而，MappingNotify事件总是被发送到所有客户端。

事件类型描述X服务器生成的特定事件。 对于每种事件类型，在X11/X.h中定义了相应的常量名称，该常量名称在引用事件类型时使用。

下表列出了事件类别及其相关的事件类型。

### Event Mask

![](https://i.loli.net/2020/04/26/ZqRWANsI2xfT9P6.png)


### XVisibilityEvent

```c++
typedef struct {
	int type;		/* VisibiltyNotify */
	unsigned long serial;	/* # of last request processed by server */
	Bool send_event;	/* true if this came from a SendEvent request */
	Display *display;	/* Display the event was read from */
	Window window;
	int state;
} XVisibilityEvent;
```

### XConfigureEvent

```c++
typedef struct {
	int type;
	/* ConfigureNotify */
	unsigned long serial;
	/* # of last request processed by server */
	Bool send_event;
	/* true if this came from a SendEvent request */
	Display *display;
	/* Display the event was read from */
	Window event;
	Window window;
	int x, y;
	int width, height;
	int border_width;
	Window above;
	Bool override_redirect;
} XConfigureEvent;
```
### XPropertyEvent

```c++
typedef struct {
	int type;
	/* PropertyNotify */
	unsigned long serial;
	/* # of last request processed by server */
	Bool send_event;
	/* true if this came from a SendEvent request */
	Display *display;
	/* Display the event was read from */
	Window window;
	Atom atom;
	Time time;
	int state;
	/* PropertyNewValue or PropertyDelete */
} XPropertyEvent;
```

### XFocusChangeEvent

```c++
typedef struct {
	int type;
	/* FocusIn or FocusOut */
	unsigned long serial;
	/* # of last request processed by server */
	Bool send_event;
	/* true if this came from a SendEvent request */
	Display *display;
	/* Display the event was read from */
	Window window;
	/* window of event */
	int mode;
	/* NotifyNormal, NotifyGrab, NotifyUngrab */
	int detail;
	/*
	* NotifyAncestor, NotifyVirtual, NotifyInferior,
	* NotifyNonlinear,NotifyNonlinearVirtual, NotifyPointer,
	* NotifyPointerRoot, NotifyDetailNone
	*/
} XFocusChangeEvent;
```

### XErrorEvent

```c++
typedef struct {
	int type;
	Display *display;
	/* Display the event was read from */
	unsigned long serial;/* serial number of failed request */
	unsigned char error_code;/* error code of failed request */
	unsigned char request_code;/* Major op-code of failed request */
	unsigned char minor_code;/* Minor op-code of failed request */
	XID resourceid;
	/* resource id */
} XErrorEvent;
```

## Display

X窗口系统支持一个或多个包含重叠窗口或子窗口的屏幕。 屏幕是物理显示器和硬件，可以是彩色，灰度或单色。 每个显示器或工作站可以有多个屏幕。 单个X服务器可以为任意数量的屏幕提供显示服务。 具有一个键盘和一个指针（通常是鼠标）的单个用户的一组屏幕称为显示器(display)。

### Display struct

```c++
/*
 * Display datatype maintaining display specific data.
 * The contents of this structure are implementation dependent.
 * A Display should be treated as opaque by application code.
 */
typedef struct _XDisplay {
	XExtData *ext_data;	/* hook for extension to hang data */
	struct _XFreeFuncs *free_funcs; /* internal free functions */
	int fd;			/* Network socket. */
	int conn_checker;         /* ugly thing used by _XEventsQueued */
	int proto_major_version;/* maj. version of server's X protocol */
	int proto_minor_version;/* minor version of servers X protocol */
	char *vendor;		/* vendor of the server hardware */
        XID resource_base;	/* resource ID base */
	XID resource_mask;	/* resource ID mask bits */
	XID resource_id;	/* allocator current ID */
	int resource_shift;	/* allocator shift to correct bits */
	XID (*resource_alloc)(); /* allocator function */
	int byte_order;		/* screen byte order, LSBFirst, MSBFirst */
	int bitmap_unit;	/* padding and data requirements */
	int bitmap_pad;		/* padding requirements on bitmaps */
	int bitmap_bit_order;	/* LeastSignificant or MostSignificant */
	int nformats;		/* number of pixmap formats in list */
	ScreenFormat *pixmap_format;	/* pixmap format list */
	int vnumber;		/* Xlib's X protocol version number. */
	int release;		/* release of the server */
	struct _XSQEvent *head, *tail;	/* Input event queue. */
	int qlen;		/* Length of input event queue */
	unsigned long request;	/* sequence number of last request. */
	char *last_req;		/* beginning of last request, or dummy */
	char *buffer;		/* Output buffer starting address. */
	char *bufptr;		/* Output buffer index pointer. */
	char *bufmax;		/* Output buffer maximum+1 address. */
	unsigned max_request_size; /* maximum number 32 bit words in request*/
	struct _XrmHashBucketRec *db;
	int (*synchandler)();	/* Synchronization handler */
	char *display_name;	/* "host:display" string used on this connect*/
	int default_screen;	/* default screen for operations */
	int nscreens;		/* number of screens on this server*/
	Screen *screens;	/* pointer to list of screens */
	unsigned long motion_buffer;	/* size of motion buffer */
	unsigned long flags;	/* internal connection flags */
	int min_keycode;	/* minimum defined keycode */
	int max_keycode;	/* maximum defined keycode */
	KeySym *keysyms;	/* This server's keysyms */
	XModifierKeymap *modifiermap;	/* This server's modifier keymap */
	int keysyms_per_keycode;/* number of rows */
	char *xdefaults;	/* contents of defaults from server */
	char *scratch_buffer;	/* place to hang scratch buffer */
	unsigned long scratch_length;	/* length of scratch buffer */
	int ext_number;		/* extension number on this display */
	struct _XExten *ext_procs; /* extensions initialized on this display */
	/*
	 * the following can be fixed size, as the protocol defines how
	 * much address space is available.
	 * While this could be done using the extension vector, there
	 * may be MANY events processed, so a search through the extension
	 * list to find the right procedure for each event might be
	 * expensive if many extensions are being used.
	 */
	Bool (*event_vec[128])();  /* vector for wire to event */
	Status (*wire_vec[128])(); /* vector for event to wire */
	KeySym lock_meaning;	   /* for XLookupString */
	struct _XLockInfo *lock;   /* multi-thread state, display lock */
	struct _XInternalAsync *async_handlers; /* for internal async */
	unsigned long bigreq_size; /* max size of big requests */
	struct _XLockPtrs *lock_fns; /* pointers to threads functions */
	/* things above this line should not move, for binary compatibility */
	struct _XKeytrans *key_bindings; /* for XLookupString */
	Font cursor_font;	   /* for XCreateFontCursor */
	struct _XDisplayAtoms *atoms; /* for XInternAtom */
	unsigned int mode_switch;  /* keyboard group modifiers */
	struct _XContextDB *context_db; /* context database */
	Bool (**error_vec)();      /* vector for wire to error */
	/*
	 * Xcms information
	 */
	struct {
	   XPointer defaultCCCs;  /* pointer to an array of default XcmsCCC */
	   XPointer clientCmaps;  /* pointer to linked list of XcmsCmapRec */
	   XPointer perVisualIntensityMaps;
				  /* linked list of XcmsIntensityMap */
	} cms;
	struct _XIMFilter *im_filters;
	struct _XSQEvent *qfree; /* unallocated event queue elements */
	unsigned long next_event_serial_num; /* inserted into next queue elt */
	int (*savedsynchandler)(); /* user synchandler when Xlib usurps */
} Display;
```


## root window

X服务器中的所有窗口都按严格的层次结构排列。 每个层次结构的顶部是一个根窗口，它覆盖了每个显示屏幕。 每个根窗口都被子窗口部分或完全覆盖。 除根窗口外，所有窗口都具有父级。 每个应用程序通常至少有一个窗口。 子窗口可能又有自己的子窗口。 这样，应用程序可以在每个屏幕上创建任意深的树。 X为窗口提供了图形，文本和光栅操作。

## Functions

### XCreateWindow 

```c++
Window XCreateWindow(Display *display, Window parent, int x, int y, unsigned int width, unsigned int height, unsigned int border_width, int depth, unsigned int class, Visual *visual, unsigned long valuemask, XSetWindowAttributes *attributes);

Window XCreateSimpleWindow(Display *display, Window parent, int x, int y, unsigned int width, unsigned int height, unsigned int border_width, unsigned long border, unsigned long background);

```

XCreateWindow函数为指定的父窗口创建一个未映射的子窗口，返回所创建窗口的窗口ID，并使X服务器生成一个CreateNotify事件。创建的窗口按照兄弟窗口的堆叠顺序放在顶部。


### XMapWindow

```c++
nt XMapWindow(Display *display, Window w);

int XMapRaised(Display *display, Window w);
int XMapSubwindows(Display *display, Window w);
```
XMapWindow函数映射具有映射请求的窗口及其所有子窗口。映射具有未映射的祖先的窗口不会显示窗口，但将其标记为在祖先映射时符合显示条件。这样的窗口称为不可查看的。当它的所有祖先都被映射后，该窗口就成为可见的，如果它没有被另一个窗口遮挡，就会在屏幕上可见。如果窗口已经映射，则此函数无效。


如果窗口的override-redirect为False，并且其他客户端在父窗口上选择了SubstructureRedirectMask，那么X服务器将生成一个MapRequest事件，而XMapWindow函数不会映射窗口。否则，窗口将被映射，X服务器将生成一个MapNotify事件。

如果窗口变得可见，并且不记得先前的内容，则X服务器将窗口与其背景平铺。如果窗口的背景未定义，则不会更改现有的屏幕内容，并且X服务器生成零或多个Expose事件。如果在未映射窗口时维护了备份存储，则不会生成任何Expose事件。如果现在将维护备份库，则始终生成全窗口曝光。否则，只能报告可见区域。类似的贴瓦和曝光发生在任何新出现的可观察的下级上。

如果窗口是一个InputOutput窗口，那么XMapWindow在它导致显示的每个InputOutput窗口上生成Expose事件。如果客户端映射并绘制窗口，并且如果客户端开始处理事件，则窗口将被绘制两次。为了避免这种情况，首先请求公开事件，然后映射窗口，以便客户机像往常一样处理输入事件。事件列表将包括屏幕上出现的每个窗口的Expose。客户机对Expose事件的正常响应应该是重新绘制窗口。这种方法通常导致更简单的程序，并与窗口管理器进行适当的交互。

### XStoreName

```c++
oid XSetWMName(Display *display, Window w, XTextProperty *text_prop);

Status XGetWMName(Display *display, Window w, XTextProperty *text_prop_return);

int XStoreName(Display *display, Window w, char *window_name);

Status XFetchName(Display *display, Window w, char **window_name_return);

```

XStoreName函数将传递给窗口名的名称分配给指定的窗口。窗口管理器可以将窗口名称显示在某些显著位置，例如标题栏，以便用户方便地识别窗口。一些窗口管理器可能会在窗口的图标中显示窗口的名称，但如果应用程序提供了窗口的图标名称，则鼓励它们使用该窗口的图标名称。如果字符串不在主机可移植字符编码中，则结果依赖于实现。 

### XSelectInput

```c++
int XSelectInput(Display *display, Window w, long event_mask);
```

XSelectInput函数请求X服务器报告与指定事件掩码关联的事件。最初，X不会报告任何这些事件。事件是相对于窗口报告的。如果一个窗口对设备事件不感兴趣，它通常会传播到最近的感兴趣的祖先，除非不传播掩码禁止它。

设置窗口的event-mask属性会覆盖对同一窗口的所有先前调用，但不会覆盖对其他客户端的调用。使用以下限制，多个客户端可以在同一个窗口中选择相同的事件。

- 多个客户端可以在同一个窗口中选择事件，因为它们的事件掩码是不相交的。当X服务器生成一个事件时，它将它报告给所有感兴趣的客户端。
- 一次只有一个客户端可以选择CirculateRequest、ConfigureRequest或MapRequest事件，这些事件与事件掩码子结构eredirectmask相关联。
- 一次只有一个客户端可以选择ResizeRequest事件，它与事件掩码ResizeRedirectMask相关联。
- 一次只能有一个客户端可以选择一个ButtonPress事件，该事件与事件掩码ButtonPressMask相关联。

服务器将事件报告给所有感兴趣的客户端。

XSelectInput会生成一个BadWindow错误。


### XDestroyWindow

```c++
int XDestroyWindow(Display *display, Window w);

int XDestroySubwindows(Display *display, Window w);

```
XDestroyWindow函数销毁指定的窗口及其所有子窗口，并使X服务器为每个窗口生成一个DestroyNotify事件。不应该再次引用窗口。如果w参数指定的窗口被映射，它将自动取消映射。DestroyNotify事件的顺序是这样的:对于任何给定的被销毁的窗口，在生成窗口本身之前，都会在窗口的任何下级上生成DestroyNotify。兄弟之间和子层次结构之间的顺序不受其他约束。如果您指定的窗口是根窗口，则不会破坏任何窗口。销毁映射窗口将在被销毁的窗口隐藏的其他窗口上生成Expose事件。

### XReparentWindow

```c++
int XReparentWindow(Display *display, Window w, Window parent, int x, int y);

```

如果指定的窗口被映射，XReparentWindow将自动对其执行UnmapWindow请求，将其从当前层次结构中的位置移除，并将其作为指定父窗口的子窗口插入。相对于同级窗口，窗口按堆叠顺序放在顶部。

### XUnmapWindow

```c++
int XUnmapWindow(Display *display, Window w);

int XUnmapSubwindows(Display *display, Window w);
``` 
XUnmapWindow函数取消对指定窗口的映射，并导致X服务器生成一个UnmapNotify事件。如果指定的窗口已经取消映射，则XUnmapWindow无效。对原来被遮挡的窗户进行正常的曝光处理。任何子窗口将不再可见，直到在父窗口上执行另一个映射调用。换句话说，子窗口仍然被映射，但是在父窗口被映射之前是不可见的。取消对窗口的映射将在原来被它隐藏的窗口上生成Expose事件。

### XRaiseWindow

```c++
int XRaiseWindow(Display *display, Window w);

       int XLowerWindow(Display *display, Window w);

       int XCirculateSubwindows(Display *display, Window w, int direction);

       int XCirculateSubwindowsUp(Display *display, Window w);

       int XCirculateSubwindowsDown(Display *display, Window w);

       int XRestackWindows(Display *display, Window windows[], int nwindows);
```


XRaiseWindow函数将指定的窗口提升到堆栈的顶部，这样就不会有其他窗口遮挡它。如果窗户被认为是叠在桌子上的纸张，那么打开窗户就相当于把纸移到堆栈的顶部，而把它的x和y位置留在桌子上不变。启动映射窗口可能会为该窗口和任何以前被隐藏的映射子窗口生成公开事件。

如果窗口的override-redirect属性为False，并且其他客户端在父节点上选择了SubstructureRedirectMask，那么X服务器将生成一个ConfigureRequest事件，并且不进行任何形式的处理。否则，窗口将被打开。

### XFlush

```c++
int XFlush(Display *display);

int XSync(Display *display, Bool discard);
int XEventsQueued(Display *display, int mode);
int XPending(Display *display);
```

XFlush函数刷新输出缓冲区。大多数客户机应用程序不需要使用这个函数，因为对XPending、XNextEvent和XWindowEvent的调用会根据需要自动刷新输出缓冲区。由服务器生成的事件可以加入到库的事件队列中。


### XParseColor

```c++
colormap, XColor *def_in_out);

int XQueryColors(Display *display, Colormap colormap, XColor defs_in_out[], int ncolors);
Status XLookupColor(Display *display, Colormap colormap, char *color_name, XColor *exact_def_return, XColor *screen_def_return);
Status XParseColor(Display *display, Colormap colormap, char *spec, XColor *exact_def_return);
```

XParseColor函数根据与指定的colormap关联的屏幕查找颜色的字符串名称。它返回确切的颜色值。如果颜色名称不在宿主可移植字符编码中，则结果依赖于实现。使用大写或小写并不重要。如果解析了名称，XParseColor返回非0;否则，它返回零。XLookupColor和XParseColor会生成 badColor 错误。

### XAllocColor

```c++
Status XAllocColor(Display *display, Colormap colormap, XColor *screen_in_out);

Status XAllocNamedColor(Display *display, Colormap colormap, char *color_name, XColor *screen_def_return, XColor *exact_def_return);
Status XAllocColorCells(Display *display, Colormap colormap, Bool contig, unsigned long plane_masks_return[], unsigned int nplanes, unsigned long pixels_return[], unsigned int npixels);
Status XAllocColorPlanes(Display *display, Colormap colormap, Bool contig, unsigned long pixels_return[], int ncolors, int nreds, int ngreens, int nblues, unsigned long *rmask_return, unsigned long *gmask_return, unsigned long *bmask_return);
int XFreeColors(Display *display, Colormap colormap, unsigned long pixels[], int npixels, unsigned long planes);

```

XAllocColor函数分配一个只读的colormap条目，对应于硬件支持的最接近的RGB值。XAllocColor返回与硬件支持的指定RGB元素最接近的颜色像素值，并返回实际使用的RGB值。对应的colormap单元格是只读的。此外，如果XAllocColor成功，则返回非零;如果失败，则返回零。可以为请求相同有效RGB值的多个客户机分配相同的只读条目，从而允许共享条目。当最后一个客户端释放一个共享单元时，它被释放。XAllocColor不使用或影响XColor结构中的标志。
XAllocColor会产生 badColor 错误。

### XSetWindowBackground

```c++
nt XChangeWindowAttributes(Display *display, Window w, unsigned long valuemask, XSetWindowAttributes *attributes);

int XSetWindowBackground(Display *display, Window w, unsigned long background_pixel);
int XSetWindowBackgroundPixmap(Display *display, Window w, Pixmap background_pixmap);
int XSetWindowBorder(Display *display, Window w, unsigned long border_pixel);
int XSetWindowBorderPixmap(Display *display, Window w, Pixmap border_pixmap);
int XSetWindowColormap(Display *display, Window w, Colormap colormap);
```
根据valuemask, XChangeWindowAttributes函数使用XSetWindowAttributes结构中的窗口属性来更改指定的窗口属性。改变背景并不会导致窗口内容的改变。要重新绘制窗口及其背景，使用XClearWindow。设置边框或更改背景，使边框的平铺原点更改导致重新绘制边框。将根窗口的背景更改为None或ParentRelative将恢复默认的背景像素图。将根窗口的边框更改为CopyFromParent将恢复默认的边框像素图。改变win-gravity不会影响窗口的当前位置。将隐藏窗口的后置存储更改为when或Always，或更改映射窗口的后置平面、后置像素或save-under，可能不会立即产生效果。更改窗口的colormap(即定义一个新映射，而不更改现有映射的内容)将生成一个ColormapNotify事件。更改可见窗口的colormap可能不会立即对屏幕产生影响，因为可能没有安装该映射(请参阅XInstallColormap)。将根窗口的光标更改为None将恢复默认光标。只要有可能，我们鼓励您共享colormap。

### XNextEvent

```c++
nt XNextEvent(Display *display, XEvent *event_return);

int XPeekEvent(Display *display, XEvent *event_return);
int XWindowEvent(Display *display, Window w, long event_mask, XEvent *event_return);
Bool XCheckWindowEvent(Display *display, Window w, long event_mask, XEvent *event_return);
int XMaskEvent(Display *display, long event_mask, XEvent *event_return);
Bool XCheckMaskEvent(Display *display, long event_mask, XEvent *event_return);
Bool XCheckTypedEvent(Display *display, int event_type, XEvent *event_return);
Bool XCheckTypedWindowEvent(Display *display, Window w, int event_type, XEvent *event_return);
```

XNextEvent函数将事件队列中的第一个事件复制到指定的XEvent结构中，然后将其从队列中删除。如果事件队列为空，XNextEvent将刷新输出缓冲区并阻塞，直到接收到事件为止。

> Region的D语言实现
> >	```d
	struct Box{
    		short x1, x2, y1, y2;
	}
	struct _XRegion {
	    c_long size;
	    c_long numRects;
	    BOX* rects;
	    BOX  extents;
	}
	alias _XRegion REGION;
> >

### XCreateRegion

```c++
Region XCreateRegion(void);

int XSetRegion(Display *display, GC gc, Region r);
int XDestroyRegion(Region r);
```

XCreateRegion函数创建一个新的空区域。

### XUnionRectWithRegion

```c++
nt XIntersectRegion(Region sra, Region srb, Region dr_return);

int XUnionRegion(Region sra, Region srb, Region dr_return);
int XUnionRectWithRegion(XRectangle *rectangle, Region src_region, Region dest_region_return);
int XSubtractRegion(Region sra, Region srb, Region dr_return);
int XXorRegion(Region sra, Region srb, Region dr_return);
int XOffsetRegion(Region r, int dx, int dy);
int XShrinkRegion(Region r, int dx, int dy);

```
XUnionRectWithRegion函数从指定矩形和指定源区域的并集更新目标区域。

XSubtractRegion函数从sra中减去srb并将结果存储在dr_return中。


### XShapeCombineRegion

```c++
void XShapeCombineRegion (
	Display *dpy,
	Window dest,
	int destKind,
	int xOff,
	int yOff,
	struct _XRegion *r,
	int op);

Operations:

ShapeSet
ShapeUnion
ShapeIntersect
ShapeSubtract
ShapeInvert
Shape Kinds:
ShapeBounding
ShapeClip
Event defines:
ShapeNotifyMask
ShapeNotify
```


## Others

XRectangle

```c++
typedef struct {
               short x, y;
               unsigned short width, height;
       } XRectangle;
```

## 参考

[Linux man pages](https://linux.die.net/man/)