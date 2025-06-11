#C_Opengl_teapot REPORT


```

// C 언어로 OpenGL을 사용하여 회전하는 주전자 출력
// NuGet 또는 수동 설치로 freeglut 설치 필요

#include <windows.h>
#include <GL/freeglut.h> // freeglut 헤더 포함
#pragma comment(lib, "freeglut.lib") // 링커에 freeglut 라이브러리 연결

float rotationAngle = 0.0f; // 주전자의 회전 각도 (애니메이션용)

// 초기화 함수: 배경색, 조명 설정 등
void init() {
    glClearColor(0.0f, 0.3f, 0.3f, 0.5f); // 배경색 설정 (청록색 계열)
    glEnable(GL_DEPTH_TEST); // 깊이 테스트 활성화 (3D 표현을 위해)

    // 조명 설정
    glEnable(GL_LIGHTING);  // 조명 기능 켜기
    glEnable(GL_LIGHT0);    // 기본 광원(0번) 사용

    // 광원의 위치 설정 (장면의 오른쪽 위)
    GLfloat light_pos[] = { 1.0f, 1.0f, 1.0f, 1.0f };
    glLightfv(GL_LIGHT0, GL_POSITION, light_pos);

    // 재질색을 glColor로 지정 가능하도록 설정
    glEnable(GL_COLOR_MATERIAL);
}

// 주전자 그리는 함수
void drawTeapot() {
    glPushMatrix(); // 현재 변환 행렬 스택에 저장

    // 회전 변환 (Y축과 Z축을 기준으로 회전)
    glRotatef(rotationAngle, 0.0f, 1.0f, 0.3f);

    // 주전자 색상 설정 (연두색)
    glColor3f(0.8f, 1.2f, 0.2f);

    // 고체 주전자 그리기 (크기 0.5)
    glutSolidTeapot(0.5);

    glPopMatrix(); // 이전 행렬로 복원 (회전 초기화)
}

// 화면을 그리는 콜백 함수
void display() {
    // 색상 버퍼와 깊이 버퍼 초기화
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    // 모델뷰 행렬 모드로 전환 후 초기화
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();

    // 카메라 시점 설정 (eye, center, up 벡터)
    gluLookAt(
        2.0, 2.5, 2.3,   // 카메라 위치 (eye)
        0.0, 0.0, 0.0,   // 카메라가 바라보는 지점 (center)
        0.0, 1.0, 0.0    // 카메라의 위쪽 방향 (up)
    );

    // 주전자 그리기
    drawTeapot();

    // 버퍼 교체 (더블 버퍼링)
    glutSwapBuffers();
}

// 창 크기 변경 시 호출되는 함수
void reshape(int w, int h) {
    if (h == 0) h = 1; // 높이가 0이 되지 않도록 보호

    // 뷰포트 설정: 창 전체를 그리기 영역으로 사용
    glViewport(0, 0, w, h);

    // 투영 행렬 설정 (원근 투영)
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0, (double)w / (double)h, 1.0, 10.0); // 시야각, 종횡비, near, far

    glMatrixMode(GL_MODELVIEW); // 다시 모델뷰로 전환
}

// 애니메이션 업데이트 함수 (타이머 콜백)
void update(int value) {
    rotationAngle += 1.0f; // 회전 각도 증가
    if (rotationAngle >= 360.0f)
        rotationAngle -= 360.0f; // 360도를 넘지 않도록 조절

    glutPostRedisplay(); // 다시 그리기 요청
    glutTimerFunc(16, update, 0); // 약 60 FPS로 타이머 재등록
}

// 메인 함수 (프로그램 시작점)
int main(int argc, char** argv) {
    glutInit(&argc, argv); // GLUT 초기화
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH); // 더블 버퍼 + RGB + 깊이버퍼
    glutInitWindowSize(800, 600); // 창 크기 설정
    glutCreateWindow("Rotating Teapot"); // 창 생성

    init(); // 초기 설정

    // 콜백 함수 등록
    glutDisplayFunc(display); // 화면 그리기
    glutReshapeFunc(reshape); // 창 크기 변경
    glutTimerFunc(16, update, 0); // 애니메이션 타이머 설정

    glutMainLoop(); // 이벤트 루프 시작
    return 0;
}

```

