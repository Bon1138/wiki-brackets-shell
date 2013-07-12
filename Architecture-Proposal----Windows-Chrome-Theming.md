# Window Wrappers

The first part of the process is to move everything to windowing objects (objectifying the windowing code).  This is basically adding C++ wrappers (analogous to the MFC construct: CWnd) around all HWND objects.

The primary reason for this is that a lot of data needs to be held on to like Fonts, Brushes and Pens as well as state data used for drawing. It's far more efficient to create the GDI objects once and use them as needed rather than creating them on demand.  Additionally, wrapping the window objects with C++ classes aids in sub-classing child windows which we'll need to do for menus (see below) and it improves the code's readability.  It also makes debugging a little easier.

Here is the base wrapper class for all windows that are created:

    class cef_window
    {
    public:
        cef_window(void);
        virtual ~cef_window(void);

        static cef_window* Create(LPCTSTR szClass, LPCTSTR szWindowTitle, DWORD dwStyles, int x, int y, int width, int height, cef_window* parent = NULL, cef_menu* menu = NULL);
        bool SubclassWindow (HWND hWnd);

        virtual LRESULT WindowProc(UINT message, WPARAM wParam, LPARAM lParam);
        LRESULT DefaultWindowProc(UINT message, WPARAM wParam, LPARAM lParam);

	    HMENU GetMenu() 
	    { return ::GetMenu(m_hWnd); }

	    BOOL UpdateWindow()
	    { return ::UpdateWindow(m_hWnd); }

	    BOOL GetWindowPlacement(LPWINDOWPLACEMENT wp)
	    { return ::GetWindowPlacement(m_hWnd, wp); }

	    BOOL SetWindowPlacement(LPWINDOWPLACEMENT wp)
	    { return ::SetWindowPlacement(m_hWnd, wp); }

	    BOOL GetWindowRect(LPRECT r)
	    { return ::GetWindowRect(m_hWnd, r); }

	    BOOL GetClientRect(LPRECT r)
	    { return ::GetClientRect(m_hWnd, r); }

	    HDC BeginPaint(PAINTSTRUCT* ps)
	    { return ::BeginPaint(m_hWnd, ps); }

	    BOOL EndPaint(PAINTSTRUCT* ps) 
	    { return ::EndPaint(m_hWnd, hdc); }

	    BOOL DestroyWindow()
	    { return ::DestroyWindow(m_hWnd); }

    protected:
        HWND m_hWnd;
        WNDPROC m_superWndProc;

	    BOOL HandleNonClientDestroy();
	    virtual void PostNonClientDestory();
    };

The wrapper is fairly similar to the MFC CWnd implementation. However, there is no message map and only the functions which we need have been implemented -- not the entire Windows API.  

NOTE: The create function takes a cef_menu* as an argument.  While we don't use this parameter it's just a noop for now.  The cef_menu class, however, will be created for menu specialization (see below).

# Message Handling
Message handling is complex and the way MFC implements message maps is fairly convoluted and tricky so the simple approach is to just have message handlers implemented in the spec classes and handle base class overloading as needed.  So far there hasn't been any need for this.  The other thing that isn't implemented in my lightweight implementation is typing of parameters and results to and from message handlers. 

MFC implements "default" handlers for many window messages. This is basically done to templatize the window message protocol and make it easier to inherit and specialize behavior.  Part of the process that MFC undergoes to route window messages casts WPARAM, LPARAM and LRESULT to and from Windows, Native and MFC types.  This is nice but somewhat difficult and expensive to implement.  To keep it simple and lightweight, there is no message map, no complex message routing architecture and no casting of parameters or results. This means that a more brute-force method of message dispatching must be implemented. 

All messages are handled in the window object's WindowProc method and each specialization class will call on class handlers. Unhandled messages are handled by the parent class' implementation by way of calling the base class' WindowProc after all message cracking is done.  If the base cef_window class implementation doesn't handle the message the message is passed to the subclassed window's window proc.

Parameter and result casting can done when the message is handled in the WindowProc method before calling the message handler for readability and ease of use.  

This is all fairly similar to what the shell does now except there is too much code inside the switch statement.

Here is the WindowProc code for the main window:

    LRESULT cef_main_window::WindowProc(UINT message, WPARAM wParam, LPARAM lParam)
    {
	    switch (message) 
	    {
	    case WM_CREATE:
		    if (HandleCreate())
			    return 0L;
		    break;
	    case WM_ERASEBKGND:
		    if (HandleEraseBackground())
			    return 0L;
		    break;
	    case WM_SETFOCUS:
		    if (HandleSetFocus())
			    return 0L;
		    break;
	    case WM_PAINT:
		    if (HandlePaint())
			    return 0L;
		    break;
	    case WM_GETMINMAXINFO:
		    if (HandleGetMinMaxInfo((LPMINMAXINFO) lParam))
			    return 0L;
		    break;
	    case WM_DESTROY:
		    if (HandleDestroy())
			    return 0L;
		    break;
	    case WM_CLOSE:
		    if (HandleClose())
			    return 0L;
		    break;
	    case WM_SIZE:
		    if (HandleSize(wParam == SIZE_MINIMIZED))
			    return 0L;
		    break;
	    case WM_INITMENUPOPUP:
		    if (HandleInitMenuPopup((HMENU)wParam)
			    return 0L;
		    break;
	    case WM_COMMAND:
		    if (HandleCommand(LOWORD(wParam)))
			    return 0L;
		    break;
	    }

        return cef_window::WindowProc(message, wParam, lParam);
    }

This is just a simple switch as we did in the old days of Windows C++ programming before there was MFC.

*[Glenn] We should try to keep the general structure of the code in cefclient_win.cpp and client_handler_win.cpp in order to make future cefclient updates as smooth as possible. I'm definitely in favor of cleaning up the code (yes, that switch statement is way too long!), but we shouldn't stray too far from the original structure.*
**_glenn, that's a good note.  I'll try to make that happen but it may be a merge of everything into one file at the end _**
  
# Subclassing 

MFC Maintains a single message pump for each thread that dispatches messages to the appropriate window through a CWndToObjMap.   

We need a similar way to map a window object to an HWND.  To do this we will use a window property. Windows implements a window property getter and setter how we'll dereference Window Object pointers:

     HANDLE GetProp (HWND, WCHAR*); 
     BOOL SetProp (HWND, WCHAR*, HANDLE);

A static window proc handles all incoming window messages before they are passed onto the C++ wrapper:

    static LRESULT CALLBACK _WindowProc (HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
    {
      cef_window* window = (cef_window*)::GetProp(hWnd, _T("CefClientWindowPtr"));
      if (window) 
      {
          return window->WindowProc(message, wParam, lParam);
      } 
      else 
      {
          return DefWindowProc(hWnd, message, wParam, lParam);
      }
    }

This is much like AfxWindowProc except it doesn't look up the CWnd object from a map. This also means that HWND objects need to be wrapped at time of creation or we won't get a WM_CREATE message.  Therefore the hook process looks something like this:

    cef_window* cef_window::Create(LPCTSTR szClassname, LPCTSTR szWindowTitle, DWORD dwStyles, int x, int y, int width, int height, cef_window* parent = NULL, CefClientMenu* menu = NULL)
    {
  
        cef_window* window = new cef_window();

        ::_HookWindowCreate(window);

        HWND hWnd = CreateWindow(szClassname, szWindowTitle, 
                                  dwStyles, x, y, width, height, hWndParent, hMenu, hInstance, (LPARAM)window);


        ::_UnHookWindowCreate();
     }

The `HookWindowCreate()` call just sets up a CBTHOOK and waits for a `HCBT_CREATEWND` message then subclasses the window.  This is almost exactly what MFC does but I added a few extras like making sure that the cef_window pointer matches the user data parameter that was passed to `CreateWindow`.  This extra safety check isn't really necessary but should give us some peace of mind:

    static LRESULT CALLBACK _HookProc(int code, WPARAM wParam, LPARAM lParam)
    {
        if (code != HCBT_CREATEWND)
            return CallNextHookEx(_HookData.m_hOldHook, code, wParam, lParam);
    
        LPCREATESTRUCT lpcs = ((LPCBT_CREATEWND)lParam)->lpcs;

        if (lpcs->lpCreateParams == (LPVOID)_HookData.m_window) 
        {
            HWND hWnd = (HWND)wParam;
            g_HookData.m_window->SubclassWindow(hWnd);
            g_HookData.Reset();
        }

        return CallNextHookEx(_HookData.m_hOldHook, code, wParam, lParam);
    }

    static void _HookWindowCreate(cef_window* window)
    {
        if (_HookData.m_hOldHook || _HookData.m_window) 
            return;

        g_HookData.m_hOldHook = ::SetWindowsHookEx(WH_CBT, _HookProc, NULL, ::GetCurrentThreadId());
        g_HookData.m_window = window;
    }

# Titlebars, System Icons and Stuff
This is pretty simple to do.  I implemented this in Dreamweaver CS5 for MDI document windows. Dreamweaver didn't use OWL documents and I wanted that "Gray" OWL look on document windows so I implemented custom drawing code to make the document frames "Gray" and customize the system buttons (Min/Max/Close).  

We need to handle all of the NC mouse messages so we know when the user is interacting with a button. This means drawing the buttons in their various states and overriding the default Windows implementation.

We'll need to setup the window to receive a `WM_NCMOUSELEAVE` message from the on WM_NCMOUSEDOWN` handler when we get a hit test on one of the system buttons.  

    void cef_main_window::TrackMouseEvents(bool track/*=true*/) 
    {
	    TRACKMOUSEEVENT tme ;
	    ::ZeroMemory(&tme, sizeof (tme));

	    tme.cbSize = sizeof (tme);

	    tme.dwFlags = TME_QUERY;
	    tme.hwndTrack = m_hWnd;
	 
	    ::TrackMouseEvent (&tme);

	    /// i.e. if we're currently tracking and the caller
	    //	 wanted to track or if we're not currently tracking and
	    //	 the caller wanted to turn off tracking then just bail
	    if ((( tme.dwFlags & TME_LEAVE ) == TME_LEAVE ) == track ) 
                 return; // nothing to do...

	    tme.dwFlags = TME_LEAVE|TME_NONCLIENT;

	    if (!track) 
		  tme.dwFlags |= TME_CANCEL;

	    // The previous call to TrackMouseEvent destroys the hwndTrack 
	    //	and cbSize members so we have to re-initialize them.
	    tme.hwndTrack = m_hWnd;
	    tme.cbSize = sizeof (tme);

	    ::TrackMouseEvent (&tme);
    }

Next add a handler to handle the WM_NCMOUSELEAVE message and dispatch it from WindowProc:

    void cef_main_window::OnNcMouseLeave() 
    {
 	    switch (m_nonClientData.m_HtActiveButton)
	    {
	        case HTCLOSE:
		case HTMAXBUTTON:
		case HTMINBUTTON:
		    m_nonClientData.m_HtActiveButton = HTNOWHERE;
		    m_nonClientData.m_bButtonOver = true ;
		    m_nonClientData.m_bButtonDown = false ;
		    DrawNonClientControlPart ( NULL, true ) ;
		    break;
	    }

	    m_nonClientData.Reset() ;
	    TrackMouseEvents( false ) ;
    }

And add some tracking data to the window object that needs it:

    class cef_main_window_non_client_data
    {
    public:
	    cef_main_window_non_client_data() 
	    {
		Reset() ;
	    }

	    void Reset ( void ) 
	    {
		m_HtActiveButton = 0 ;
		m_bButtonDown = false ;
    	        m_bButtonOver = false ;
	    }

	    UINT m_HtActiveButton;
	    bool m_bButtonDown ;
	    bool m_bButtonOver ;
    };

# Menus
Menus are new because I haven't done much with them yet.  I started experimenting with this in my Reflow Innovation project and got some results but not as dramatic as what was laid out in the XD spec.  After reading Peter Flynn's comment about needing a minimum, I'm a little concerned about putting the menus in the title bar.  _After discussions with XD, Menus in the titlebar will not be implemented_

***
**DISREGARD THE REMAINDER OF THIS SECTION**  
The discussion below is for subclassing the actual popup menu to draw the menu and menu items with a black background.  _This will also not be implemented_
***

The first step in the process is to get everything to draw a black menu bar and menu frame.  We do this by drawing the menubar ourselves in the WM_NCPAINT handler. 

In the WM_NCPAINT handler we need to iterate over all of the menu items in the menu bar and add owner drawn attributes on anything that isn't already setup for owner draw. This would be new things that are added since the last time the window was painted.

    void cef_main_window::MakeMenuBarOwnerDrawn() 
    {
            MENUBARINFO mbi;
            ZeroMemory(&mbi, sizeof(mbi));
            mbi.cbSize = sizeof(mbi);

            GetMenuBarInfo(hWnd, OBJID_MENU, 0, &mbi);

            int items = GetMenuItemCount(mbi.hMenu);

            for (int i = 0; i < items; i++) 
            {
                MENUITEMINFO mii;
                ZeroMemory(&mii, sizeof(mii));
                mii.cbSize = sizeof(mii);
                mii.fMask = MIIM_FTYPE|MIIM_DATA;

                // Make these owner drawn...
                GetMenuItemInfo(mbi.hMenu, i, TRUE, &mii);

                // Make all menu items owner drawn
                MakeMenuItemsOwnerDrawn(GetSubMenu(mbi.hMenu, i));
            }
    }

Then each menu item of each menu is made owner drawn:

    void cef_main_window::MakeMenuItemsOwnerDrawn(HMENU hmenu) 
    {
        int count = GetMenuItemCount(hmenu);
        for (int i = 0; i < count; i++)
        {
            MENUITEMINFO mii = {sizeof(mii) };
            mii.fMask = MIIM_FTYPE | MIIM_DATA;
            if (GetMenuItemInfo(hmenu, i, TRUE, &mii))
            {
                if ((mii.fType & MFT_OWNERDRAW) == 0)
                {
                    MENUITEMDATA *pmid = (MENUITEMDATA *)new MENUITEMDATA(hmenu, i);

                    mii.dwItemData = reinterpret_cast<ULONG_PTR>(pmid);
                    mii.fType |= MFT_OWNERDRAW;
                    SetMenuItemInfo(hmenu, i, TRUE, &mii);
                }
            }
        }
    }

After that then you just need to handle the WM_MEASUREITEM and WM_DRAWITEM messages.

This is as far as I got with my prototype. 

It worked but it doesn't put the menus in the title bar like Photoshop and other CS OWL based apps which is what Larz's spec calls for. If we wanted to keep the menus where they are then this will work fairly well but It'll need to be tested on different platforms and different drawing modes..  

The only thing that wasn't done was to get a black background for menus. I did a lot of experimentation and did quite a bit of research.  I have used an MFC based application theming library in the past (http://www.codejock.com/products/toolkitpro/) and, before that, implemented cool menu code from Paul DiLascia http://www.microsoft.com/msj/0198/coolmenu.aspx in various projects.  I looked at these to see how they solved some problems to glean an idea on how to solve some of the menuing issues for Brackets but cannot use the code as-is since they both depend on MFC.  I also looked at MenuXP from codeproject (http://www.codeproject.com/Articles/1809/CMenuXP-The-Office-XP-Style-Menu) -- Code Project Open License (http://www.codeproject.com/info/cpol10.aspx).  This cannot be used as a library drop-in either since it depends on MFC. 

A lot can be learned from the 3 to help solve some of the drawing issues and accessibility issues.

Popup Menus are just another window object (oddly though, they don't show up in SPY++) and can be hooked to do custom drawing like we do for any other window.  

To hook a popup menu, you need to basically hook the window create process for all windows being created on the current thread and check for the special menu window class name ("#32768").  Dialogs, BTW, have class "#32770".  These probably represent system atoms but that's another talk. Hooking all window create messages for the current process means that we can theme the system menu and system context menus.  Brackets (and Edge Code) doesn't use system context menus but  Reflow does. So this is something that the unified

*[Glenn] Native pop-up menus are in the backlog, so that is definitely something we'll need to address.*
**_I was planning to add a test menu to make sure it works in Brackets then it's just a matter of creating a popup context menu_**

So we basically re-use the window hook that we implemented for cef_window above and add another qualifier to check for menus:

        TCHAR szClassName[10];
        int Count = ::GetClassName (hWnd, szClassName, lengthof(szClassName));

        // Check for the menu-class
        if (_tcscmp (szClassName, _T("#32768")) == 0 )
        {
            ... subclass the menu
        }

Then the window object implementation needs to handle the drawing code for the menu (which is quite extensive).   

# Testing, 1-2-3
The good news is that I've done most of the work in various forms and in different parts of the universe so this is just bringing them all together.  This started as an innovation project for Reflow.  Since I've done this many times I thought it would be easy (and it was) so the challenge was to make the menus work.  I think after the research I've done that I can move forward pretty quickly but Chris Bank tried to demo my build for the Reflow innovation day for me, since I was out on sabbatical, and wasn't able to get it to work on Windows Vista.  This is a risk. It was originally developed on XP so every version (and new version) of Windows needs to be tested.  The acceptance criteria doesn't call for owner drawn dialog boxes.  There aren't any except the File Dialogs (which is another story) so this should be a fairly quick inspection on all platforms.  It's just a matter of having access to VMWare and testing on every incarnation of windows with various accessibility settings, themes and changing various window customization settings.

*[Glenn] Dialog boxes are out of scope. If XP support requires a bunch of additional work, we should consider dropping it. This is ultimately a PM decision, but engineering effort may influence it.*

# Customization
The idea is that we could turn the darkness on and off by wrapping the code into either a runtime check or compiletime flag.  Either will work -- but we haven't talked about this in detail.  Furthermore, the darkness colors will be compile time constants -- not driven by an external style sheet or config file.  This wasn't part of the acceptance criteria and I know that's what folks are asking for but that's simply too much work for this round.  I believe there is another story to allow the theme to be customizable.

*[Glenn] A global flag to enable/disable is fine. This can be compile time or runtime, whichever is easier. Compile time constants for colors is fine as a first step, but we may need to make them customizable at runtime in the future.*

NOTE: This work is for the window chrome only.  The Code Mirror colors are not changing and can be customized using Code Mirror theme files.  There is an extension to do this already but I haven't tried it.