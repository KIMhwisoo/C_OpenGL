```

// OpenGL 라이브러리를 링크에 추가
#pragma comment(lib, "opengl32.lib")

// 윈도우 API 및 OpenGL 헤더 포함
#include <windows.h>
#include <gl/GL.h>

// 윈도우 프로시저 함수: 메시지를 처리하는 콜백 함수
LRESULT CALLBACK WndProc(HWND h, UINT m, WPARAM w, LPARAM l) {
    if (m == WM_DESTROY) {
        PostQuitMessage(0); // 프로그램 종료 요청
        return 0;
    }
    return DefWindowProc(h, m, w, l); // 기본 메시지 처리
}

// WinMain: 윈도우 애플리케이션의 진입점
int WINAPI WinMain(HINSTANCE hInst, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmd) {
    // 윈도우 클래스 설정 (OpenGL용 DC를 고정하기 위해 CS_OWNDC 사용)
    WNDCLASS wc = { CS_OWNDC, WndProc, 0, 0, hInst, 0, 0, 0, 0, L"GL" };
    RegisterClass(&wc); // 클래스 등록

    // 윈도우 생성
    HWND hwnd = CreateWindow(
        L"GL", L"Quadrilateral",            // 클래스 이름, 창 제목
        WS_OVERLAPPEDWINDOW,                // 창 스타일
        100, 100, 800, 600,                 // 위치(x, y), 크기(width, height)
        0, 0, hInst, 0                      // 부모/메뉴/인스턴스/파라미터
    );
    ShowWindow(hwnd, nCmd); // 창 표시

    // 디바이스 컨텍스트 얻기 (그리기 위한 핸들)
    HDC dc = GetDC(hwnd);

    // 픽셀 포맷 지정 (OpenGL에서 사용할 수 있도록 설정)
    PIXELFORMATDESCRIPTOR pfd = {
        sizeof(pfd), 1, // 구조체 크기와 버전
        PFD_DRAW_TO_WINDOW | PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER, // 윈도우에 그리기, OpenGL 지원, 더블 버퍼 사용
        PFD_TYPE_RGBA // 색상 포맷은 RGBA
    };

    // 픽셀 포맷 설정
    SetPixelFormat(dc, ChoosePixelFormat(dc, &pfd), &pfd);

    // OpenGL 렌더링 컨텍스트 생성 및 활성화
    HGLRC rc = wglCreateContext(dc);
    wglMakeCurrent(dc, rc);

    // 4개의 꼭짓점(x, y 좌표 + RGB 색상) 데이터 정의
    float v[] = {
        //   x,     y,    r, g, b
        -0.5f,  0.5f,  1, 0, 0, // 좌측 상단 (빨강)
         0.5f,  0.5f,  0, 1, 0, // 우측 상단 (초록)
         0.5f, -0.5f,  0, 0, 1, // 우측 하단 (파랑)
        -0.5f, -0.5f,  1, 1, 0  // 좌측 하단 (노랑)
    };

    MSG msg;
    while (1) {
        // 메시지 큐에서 메시지 처리
        if (PeekMessage(&msg, 0, 0, 0, PM_REMOVE)) {
            if (msg.message == WM_QUIT) break; // 종료 요청이면 루프 종료
            TranslateMessage(&msg); // 키보드 메시지 처리
            DispatchMessage(&msg);  // 윈도우 프로시저로 전달
        }
        else {
            // 화면 초기화
            glClearColor(0.2f, 0.3f, 0.3f, 1.0f); // 배경색 설정
            glClear(GL_COLOR_BUFFER_BIT);       // 색 버퍼 지우기

            // 정점과 색상 배열 사용 활성화
            glEnableClientState(GL_VERTEX_ARRAY);
            glEnableClientState(GL_COLOR_ARRAY);

            // 정점 위치 포인터 설정 (2개씩 사용, 5개 단위로 반복)
            glVertexPointer(2, GL_FLOAT, 5 * sizeof(float), v);

            // 색상 포인터 설정 (3개씩 사용, 위치 다음 요소부터 시작)
            glColorPointer(3, GL_FLOAT, 5 * sizeof(float), v + 2);

            // 사각형 그리기 (4개의 정점 사용)
            glDrawArrays(GL_QUADS, 0, 4);

            // 더블 버퍼 교체 (백 버퍼 → 프론트 버퍼)
            SwapBuffers(dc);
        }
    }

    // 종료 시 리소스 해제
    wglMakeCurrent(0, 0);      // 현재 컨텍스트 해제
    wglDeleteContext(rc);      // OpenGL 컨텍스트 삭제
    ReleaseDC(hwnd, dc);       // 디바이스 컨텍스트 해제

    return 0; // 프로그램 종료
}
```
