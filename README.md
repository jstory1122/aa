#define UNICODE  // 이미 프로젝트에서 정의되어 있다면 이 줄을 삭제해도 됩니다.
#include <windows.h>
#include <gdiplus.h>
#include <stdio.h>

#pragma comment (lib,"gdiplus.lib")

using namespace Gdiplus;

void InitializeGDIPlus(ULONG_PTR* gdiplusToken) {
    GdiplusStartupInput gdiplusStartupInput;
    GdiplusStartup(gdiplusToken, &gdiplusStartupInput, NULL);
}

void ShutdownGDIPlus(ULONG_PTR gdiplusToken) {
    GdiplusShutdown(gdiplusToken);
}

void DisplayImage(const WCHAR* filename, HDC hdc, int x, int y) {
    Image image(filename);

    // 이미지 크기 얻기
    UINT imgWidth = image.GetWidth();
    UINT imgHeight = image.GetHeight();

    // 그리기
    Graphics graphics(hdc);
    graphics.SetSmoothingMode(SmoothingModeHighQuality);  // 고품질 렌더링 모드 설정
    graphics.DrawImage(&image, x, y, imgWidth, imgHeight);
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    static ULONG_PTR gdiplusToken;
    switch (uMsg) {
    case WM_CREATE:
        InitializeGDIPlus(&gdiplusToken);
        break;
    case WM_PAINT: {
        PAINTSTRUCT ps;
        HDC hdc = BeginPaint(hwnd, &ps);

        // 원하는 위치에 이미지를 표시
        DisplayImage(L"jcshim.jpg", hdc, 10, 10);
        DisplayImage(L"jcshim.png", hdc, 300, 10);

        EndPaint(hwnd, &ps);
        break;
    }
    case WM_DESTROY:
        ShutdownGDIPlus(gdiplusToken);
        PostQuitMessage(0);
        break;
    default:
        return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
    return 0;
}

int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PWSTR pCmdLine, int nCmdShow) {
    const WCHAR CLASS_NAME[] = L"Sample Window Class";

    WNDCLASS wc = {};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

    HWND hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        L"Display JPG and PNG",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, 600, 400,
        NULL,
        NULL,
        hInstance,
        NULL
    );

    if (hwnd == NULL) {
        return 0;
    }

    ShowWindow(hwnd, nCmdShow);

    MSG msg = {};
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}
