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

## Event

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
nt XUnmapWindow(Display *display, Window w);

int XUnmapSubwindows(Display *display, Window w);
``` 
XUnmapWindow函数取消对指定窗口的映射，并导致X服务器生成一个UnmapNotify事件。如果指定的窗口已经取消映射，则XUnmapWindow无效。对原来被遮挡的窗户进行正常的曝光处理。任何子窗口将不再可见，直到在父窗口上执行另一个映射调用。换句话说，子窗口仍然被映射，但是在父窗口被映射之前是不可见的。取消对窗口的映射将在原来被它隐藏的窗口上生成Expose事件。

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