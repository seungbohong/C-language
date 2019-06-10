~~~c

/************************************************************************
 *                                    *
 *                Tetris Program                *
 *                                    *
 ************************************************************************/
#include <ncurses.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define BLANK 0
#define WALL -1
#define FALSE 0
#define TRUE  1

#define LEVEL2_SCORE 400                        // LEVEL UP 기준 점수
#define LEVEL3_SCORE 800
#define LEVEL4_SCORE 1200
#define LEVEL5_SCORE 2000

#define LEVEL1_SPEED 800                        // LEVEL 별 속도
#define LEVEL2_SPEED 400
#define LEVEL3_SPEED 200
#define LEVEL4_SPEED 100
#define LEVEL5_SPEED 50

#define LEVEL_MESSAGE_Y 13                      // 현재 LEVEL이 찍히는 위치
#define SCORE_MESSAGE_Y 12                      // 현재 점수가 찍히는 위치

#define PREVIEW_Y 5                             // NEXT 블록이 찍히는 위치
#define PREVIEW_X 40

#define STAGEX 5                                // 게임화면의 위치
#define STAGEY 3


int Total =  0;                                 // 한줄 지웠을 때 update를 위한 변수
int Block_X, Block_Y;                           // 블록의 위치
int Block_No, Block_No2;                        // 블록의 종류
int Field[26][12];                              // 쌓인 블록, 충돌검사용
int BlockSpeed;                                 // 현재 속도
int Level_no;                                   // 현재난이도
int Select_no=1;                                // 기본 메뉴 default Select 선언

/* 테트리스 블럭 모양 */
/* 이차원배열 [4][4]에서 1들의 집합이 나타내는 것이 도형 */
int Block_Pattern[7][4][4] = {
    {
        {0,1,0,0},
        {0,1,0,0},
        {0,1,0,0},
        {0,1,0,0}
    },
    {
        {0,0,0,0},
        {0,1,1,0},
        {0,1,0,0},
        {0,1,0,0}
    },
    {
        {0,0,1,0},
        {0,1,1,0},
        {0,1,0,0},
        {0,0,0,0}
    },
    {
        {0,1,0,0},
        {0,1,1,0},
        {0,0,1,0},
        {0,0,0,0}
    },
    {
        {0,0,0,0},
        {0,1,0,0},
        {1,1,1,0},
        {0,0,0,0}
    },
    {
        {0,0,0,0},
        {0,1,1,0},
        {0,1,1,0},
        {0,0,0,0}
    },
    {
        {0,0,0,0},
        {0,1,1,0},
        {0,0,1,0},
        {0,0,1,0}
    }
};

void Draw_Game(void);
void Draw_Wall(void);
void Check_Lines(void);
void Draw_Block(void);
void Erase_Block(void);
void preview(void);
void Draw_Menu(void);
void Draw_Menu2(void);
void start(void);

int Turn_Block(void);
int Init_Game(void);
int key_in(void);
int Down_Proc(void);
int Crush_Block(int x0, int y0, int no);
int Crush_Block2(void);                         // Turn_Block용 충돌검사

/************************************************************************
 *                                    *
 *                Preview             *
 *                                    *
 ************************************************************************/
void preview(void)                              // 다음 블록을 찍는 함수
{
    int x, y;
    for(y = 0;y < 4;y++){                       // next를 초기화함
        for(x = 0; x < 4; x++){
            mvprintw(PREVIEW_Y+y, PREVIEW_X+2*x,"  ");
        }
    }
    Block_No2 = rand() % 7;                     // 0~6까지 임의의 값 저장
    attron(COLOR_PAIR(Block_No2+1));            // 1더해서 색상쌍 부여
    
    for (y = 0; y < 4; y++) {
        for (x = 0; x < 4; x++) {               // Block_No2의 도형부분을
            if (Block_Pattern[Block_No2][y][x]!=0) {
                mvaddstr (PREVIEW_Y+y,PREVIEW_X+2*x, "  ");
                                                // preview창에 찍어준다
            }
        }
    }
    attroff(COLOR_PAIR(Block_No2+1));
}  // 미리보기함

/************************************************************************
 *                                    *
 *                Preview2                *
 *                                    *
 ************************************************************************/
void preview2(void)
{
    //next 윈도우 초기화 과정을 뺸 preview. Block_No2는 함수를 호출하기 전에 값을 받는다
    int x, y;
    attron(COLOR_PAIR(Block_No2+1));
    for(y=0; y<4; y++){
        for(x=0; x<4; x++){
            if(Block_Pattern[Block_No2][y][x]!=0){
                mvaddstr(PREVIEW_Y+y, PREVIEW_X+2*x, "  ");
            }
        }
    }
}

/************************************************************************
 *                                    *
 *                Draw_Block                *
 *                                    *
 ************************************************************************/
void Draw_Block(void)
{
    int x, y,y_line;
    
    attron(COLOR_PAIR(Block_No+1));      //Block_No의 색을 키고
    for (y = 0; y < 4; y++) {
        for (x = 0; x < 4; x++) {
            if (Block_Pattern[Block_No][y][x])
                mvaddstr (Block_Y+y+STAGEY,2*(Block_X+x)+STAGEX, "  ");
            //stage 의 y값 + 블록 위치를 의미하는 Block_Y 값 + 4*4에서의 블록 위치인 y 를 통해 y값을 구함
            //같은 방법으로 x값을 구하고 그 좌표에 블록을 그린다
        }
    }
    attroff(COLOR_PAIR(Block_No+1));
}

/************************************************************************
 *                                    *
 *                Erase_Block                *
 *                                    *
 ************************************************************************/
void Erase_Block(void)
{
    int x, y;
    for (y = 0; y < 4; y++) {
        for (x = 0; x < 4; x++) {
            if (Block_Pattern[Block_No][y][x]!=0) {
                mvprintw (Block_Y+y+STAGEY,2*(Block_X+x)+STAGEX, "  ");
                //그렸던 블록의 위치에 공백을 넣어 블록을 삭제함
            }
        }
    }
}

/************************************************************************
 *                                    *
 *                Draw_Game                *
 *                                    *
 ************************************************************************/
/* 현재 Field에 쌓여져있는 block들을 출력해주는 함수 */
void Draw_Game(void)
{
    int i,j;
    for(i = 0; i<24; i++) {
        for(j=1;j<11;j++)
        {
            if(Field[i][j]!=0) {                            // Field가 비지 않았다면
                attron(COLOR_PAIR(Field[i][j]));            // 해당 Field 위치의 값에 알맞는 색상의 속성을 on
                mvprintw (i+STAGEY,2*j+STAGEX, "  ");       // 헤당 인덱스의 블록을 알맞는 색상으로 출력해줌
                attroff(COLOR_PAIR(Field[i][j]));           // 사용후에는 색상 속성을 off
            } else { mvprintw (i+STAGEY,2*j+STAGEX, "  ");} // 비었을 경우 빈칸을 출력
        }
    }
}

/************************************************************************
 *                                    *
 *                Draw_Wall                *
 *                                    *
 ************************************************************************/
/* Tetris의 Field[x][y]와 Next의 테두리를 그리는 함수 */
void Draw_Wall(void)
{
    int x, y;
    // 게임화면 테두리를 그린다.
    mvprintw(1, STAGEX+3,"[ Tetris ]");                     // 게임화면 Field화면 위에, 즉 (1, STAGEX+3)의 위치에서 [ Tetris ] 라는 문자열을 출력
    mvhline(24+STAGEY, 2+STAGEX,ACS_HLINE, 21);             // 게임화면 아래부분의 수평테두리를 그림
    mvhline(2, 1+STAGEX, ACS_HLINE, 21);                    // 게임화면 윗부분의 수평테두리를 그림
    mvvline(2, 1+STAGEX, ACS_ULCORNER, 1);                  // 게임화면 좌측상단 코너테두리를 그림
    mvvline(2, 22+STAGEX, ACS_URCORNER, 1);                 // 게임화면 우측상단 코너테두리를 그림
    mvhline(24+STAGEY, 1+STAGEX, ACS_LLCORNER, 1);          // 게임화면 좌측하단 코너테두리를 그림
    mvhline(24+STAGEY,22+STAGEX, ACS_LRCORNER, 1);          // 게임화면 우측하단 코너테두리를 그림
    mvvline(0+STAGEY, 1+STAGEX, ACS_VLINE, 24);             // 게임화면 좌측의 수직선테두리를 그림
    mvvline(0+STAGEY, 22+STAGEX, ACS_VLINE, 24);            // 게임화면 우측의 수직선 테두리를 그림
    
    // NEXT화면 테두리를 그린다.
    mvhline(PREVIEW_Y-1, PREVIEW_X-1, ACS_HLINE, 10);       // NEXT화면 상단의 수평테두리를 그림
    mvhline(PREVIEW_Y+4, PREVIEW_X-2, ACS_HLINE, 10);       // NEXT화면 하단의 수평테두리를 그림
    mvvline(PREVIEW_Y-1, PREVIEW_X-2, ACS_ULCORNER, 1);     // NEXT화면 좌측상단의 코너테두리를 그림
    mvvline(PREVIEW_Y-1, PREVIEW_X+8, ACS_URCORNER, 1);     // NEXT화면 우측상단의 코너테두리를 그림
    mvvline(PREVIEW_Y, PREVIEW_X+8, ACS_VLINE, 5);          // NEXT화면 우측의 수직선 테두리를 그림
    mvvline(PREVIEW_Y, PREVIEW_X-2, ACS_VLINE, 5);          // NEXT화면 좌측의 수직선 테두리를 그림
    mvhline(PREVIEW_Y+4, PREVIEW_X-2, ACS_LLCORNER, 1);     // NEXT화면 좌측하단의 코너테두리를 그림
    mvhline(PREVIEW_Y+4, PREVIEW_X+8, ACS_LRCORNER, 1);     // NEXT화면 우측하단의 코너테두리를 그림
    mvprintw(PREVIEW_Y-1,PREVIEW_X," NEXT: ");              // NEXT화면 위쪽의 NEXT: 라는 문자열을 출력
    mvprintw(SCORE_MESSAGE_Y, 39,"SCORE: %6d", Total);      // 현재 SCORE을 알려주는 문자열을 출력
    mvprintw(LEVEL_MESSAGE_Y, 39,"LEVEL %d", Level_no);     // 현재 LEVEL을 알려주는 문자열을 출력
}

/************************************************************************
 *                                    *
 *                Crush_Block                *
 *                                    *
 ************************************************************************/
/* 블록끼리 crush된 경우를 확인하기 위한 함수 */
int Crush_Block(int x0, int y0, int no){                    // 현재 Block의 위치와 블록의 종류를 알려주는 변수가 인자로 주어짐
    int x, y;
    
    for (y = 0; y < 4; y++) {
        for (x = 0; x < 4; x++) {                           // 현재 Block과 Field에 있는 값이 모두 1이면 같은 위치에 도형끼리 충돌한 것
            if ((Block_Pattern[no][y][x]!=0) && (0!=Field[y0+y][x0+x]))
                return TRUE;                                // 양쪽다 1이면 TRUE 리턴
        }
    }
    return FALSE;                                           // 충돌하지 않은 경우는 FALSE 리턴
    
}

/************************************************************************
 *                                    *
 *                Crush_Block2                *
 *                                    *
 ************************************************************************/
/* Turn_Block시에 블록의 충돌을 확인하는 함수 */
int Crush_Block2() {
    int x, y;
    int temp[4][4] = {0};                                   // Turn_Block된 후의 Block의 배열을 저장하기 위한 이차원 배열 temp를 초기화
    
    for(y = 0; y<4; y++) {
        for(x = 0; x<4; x++) {
            temp[y][x] = Block_Pattern[Block_No][y][x];     // 현재 Block을 temp에 대입
        }
    }
    
    for(y = 0; y<4; y++) {
        for(x = 0; x<4; x++) {                              // 반시계 방향으로 90도 회전한 도형을 temp를 이용하여 만들고 이것이 충돌했다면 TRUE를 리턴
            if (Field[Block_Y+y][Block_X+x] && temp[3-x][y])
                return TRUE;
        }
    }
    return 0;                                               // 모두 실행되고나면 0을 리턴
}

/************************************************************************
 *                                    *
 *                Turn_Block                *
 *                                    *
 ************************************************************************/
/* 현재의 Block을 반시계방향으로 90도 회전시키는 함수 */
int Turn_Block() {
    int i, j;
    int temp[4][4] = {0};                                   // 회전한 Block을 저장할 이차원 배열 temp 초기화
    for(i = 0; i<4; i++) {
        for(j = 0; j<4; j++) {
            temp[i][j] = Block_Pattern[Block_No][i][j];     // temp에 회전 이전의 블록 모양을 대입
        }
    }
    
    for(i = 0; i<4; i++) {
        for(j = 0; j<4; j++) {
            Block_Pattern[Block_No][i][j] = temp[3-j][i];   // 반시계 방향으로 90도 회전한 블록을 블록 모양에 대입
        }
    }
    return 0;
}

/************************************************************************
 *                                    *
 *                Init_Game                *
 *                                    *
 ************************************************************************/
/* 게임을 전부 초기화 시키는 함수 */
int Init_Game() {
    
    int x,y;                                                // Field 배열의 x, y좌표를 의미
    for (y = 0; y < 25; y++) {
        for (x = 0; x < 12; x++) {
            if (x == 0 || x == 11 || y == 24) {             // 게임화면의 테두리 부분은 WALL값으로 처리함
                Field[y][x] = WALL;
            } else    {                                     // 그외는 BLANK(빈칸)값으로 처리
                Field[y][x] = BLANK;
            }
        }
    }
    
    Block_X = 4;                                            // 맨 처음 down시 Block의 위치를 설정함(화면 맨위의 가운데)
    Block_Y = 0;
    
    return 0;
}

/************************************************************************
 *                                    *
 *                key_in                  *
 *                                    *
 ************************************************************************/
/* 방향키를 이용한 테트리스의 조작과 일시정지, 입력하지 않아도 블록이 떨어지는 조작을 제어해주는 함수 */
int key_in() {
    int ch;                                                 // 입력값을 저장하는 변수 ch
    ch = getch();                                           // 입력값을 대입
    switch(ch) {
        case 'p':                                           // 'p'라는 입력을 받으면 일시정지 창(Draw_Menu2)를 생성
            Draw_Menu2();
            break;
        case ' ':                                           // 스페이스값을 입력받으면
            while(1){                                       // 아래에있는 블록에 닿기전까지 계속해서
                if (!Crush_Block(Block_X, Block_Y+1, Block_No)) {
                    Erase_Block();                          // 지금 블록을 지우고
                    Block_Y++;                              // y축으로 1만큼 블록의 위치가 아래로 내려가고
                    Draw_Block();                           // 내려간 블록을 화면에 그려준다.
                } else {
                    break;
                }
            }break;
        case KEY_UP:                                        // UP키를 입력받으면
            if (!Crush_Block2()) {                          // 회전시 충돌하지 않으면
                Erase_Block();                              // 현재의 블록을 지우고
                Turn_Block();                               // 블록을 회전시켜
                Draw_Block();                               // 회전된 블록을 그려준다.
            }break;
        case KEY_DOWN:                                      // DOWN키를 입력받으면
            if (!Crush_Block(Block_X, Block_Y+1, Block_No)) {
                Erase_Block();                              // y축으로 1만큼 내려갈때 충돌되지 않는다면
                Block_Y++;                                  // 현재의 블록을 지우고 y축에서 1만큼 아래로 이동하고 이동한 블록을 그려준다.
                Draw_Block();
            }break;
        case KEY_LEFT:                                      // LEFT키를 입력받으면
            if (!Crush_Block(Block_X-1, Block_Y, Block_No)) {
                Erase_Block();                              // x축에서 왼쪽으로 1만큼 이동할때 충돌되지 않는다면
                Block_X--;
                Draw_Block();                               // 현재의 블록을 지우고 x축에서 1만큼 왼쪽으로 이동한 블록을 그린다.
            }break;
        case KEY_RIGHT:                                     // RIGHT 키를 입력받으면
            if (!Crush_Block(Block_X+1, Block_Y, Block_No)) {
                Erase_Block();                              // x축에서 오른쪽으로 1만큼 이동할때 충돌되지 않는다면
                Block_X++;
                Draw_Block();                               //  현재의 블록을 지우고 x축에서 1만큼 오른쪽으로 이동하고 이동한 블록을 그려준다.
            }break;
        default :                                           // 해당되지 않다면 break
            break;
    }                                                       // switch문 종료
    return 0;
}

/************************************************************************
 *                                    *
 *                Down_Proc                *
 *                                    *
 ************************************************************************/
int Down_Proc() {           // 자동으로 한 칸씩 밑으로 내려가는 함수
    int x, y, ch;           // 반복문에 사용할 변수 x, y와 getch로 값을 받아올 ch 변수 선언
    int main(void);
    
    if (!Crush_Block(Block_X, Block_Y+1, Block_No)) {   // Block이 충돌나지 않으면
        Erase_Block();      // 현재 블록 삭제
        Block_Y++;          // 블록을 한 칸 밑으로 내리고
        Draw_Block();       // 재귀함수 호출
    } else {                                            // Block이 충돌난다면
        for (y = 0; y < 4; y++) {                       // 이중 반복문을 통해 y는 4번 반복
            for (x = 0; x < 4; x++) {                   // x도 4번 반복하여
                if (Block_Pattern[Block_No][y][x]!=0) {         // 블록의 패턴을 변경할 수 있으면
                    Field[Block_Y+y][Block_X+x] = Block_No+1;   // 블록의 모형 변경
                }
            }
        }
        
        
        if(Block_Y<(0+STAGEY)) {                        // 블록이 표시될 수 없을 경우 (게임 종료)
            erase ();                                   // 화면을 깨끗하게 지워줌
            timeout(5000);                              // 5초 시간 경과
            mvprintw(9,13,"         ");                 // 공백 출력
            mvprintw(10,20,"GAME OVER");                // GAME OVER을 계속 화면에 띄워주는 효과
            mvprintw(12, 20, "Do you want restart(y/n)?"); // 다시 시작할 건지 출력
            while(ch = getch()){                        // 값을 입력받아
                if(ch == 'y'){                          // y일 경우
                    clear();                            // 초기화시키고
                    Total=0;                            // 점수를 0으로 만든 뒤, 빠져나옴
                    main();
                    break;
                }
                else if(ch=='n'){                       // n일 경우
                    clear();                            // 초기화시키고
                    endwin();                           // 라이브러리 처리
                    exit(0);                            // 강제 종료 함수 호출
                    break;
                }
            }
        }
        
        Check_Lines();                                  // 라인을 확인하는 함수 호출
        
        Block_X = 4;                                    // 블록의 크기인 Block_X와 Block_Y값 초기화
        Block_Y = 0;
        Block_No = Block_No2;                           // 미리보기 블록을 현재 블록에 가져옴
        
    preview();                                      // preview 함수를 통해 블록 생성
        
    }
    return 0;
}

/************************************************************************
 *                                    *
 *                Check_Lines                *
 *                                    *
 ************************************************************************/
void Check_Lines()                                  // 쌓인 블록이 x축으로 꽉 채워지는지 확인해주는 함수
{
    int i, j, k;                                    // 반복문에 사용할 변수 선언
    int comp;                                       // 현재 x축의 쌓인 블록의 갯수를 저장하는 변수
    int lines = 0;                                  // 줄을 없앤 수를 저장할 변수
    int score = 0;                                  // 점수를 저장할 변수
    int line_bonus=0;                               // 줄을 없앤 보너스를 저장할 변수 선언 및 0으로 초기화
    
    for(i = 0; i<24; i++) {                         // 벽의 Y축인 24번 반복
        comp=0;                                     // i를 증가시킬 때마다 comp를 0으로 초기화
        for(j = 1; j<11; j++) {                     // 벽의 X축인 10번 반복
            
            if(Field[i][j]) {                       // 테트리스가 해당 x축 라인에 있을 경우
                ++comp;                             // comp를 1 증가
            }
        }
        
        if(comp == 10) {                            // comp가 10이라면(x축이 꽉 찼다면)
            lines++;                                // 라인을 1 증가
            for(k = i; k>0; k--) {                  // x축을 기준으로 한 줄을 삭제
                for(j=1;j<11;j++) {
                    Field[k][j] = Field[k-1][j];
                }
            }
            ++Block_Y;                              // 전체 블록을 한 칸 밑으로 내림
        }//if종료
    }
    
    
    switch(lines) {                                 // 한 번에 라인을 없앤 수에 따라 보너스 점수 획득
        case 2:                                     // 2줄 없애면 200점 추가
            line_bonus=200;
            break;
        case 3:                                     // 3줄 없애면 400점 추가
            line_bonus=400;
            break;
        case 4:                                     // 4줄 없애면 600점 추가
            line_bonus=600;
            break;
        case 5:                                     // 5줄 없애면 800점 추가
            line_bonus=800;
            
    }
    
    score=lines*100+line_bonus;                         // 없앤 라인 수는 100씩 부여하고, 라인 보너스를 더해 score 변수에 저장
    Total += score;                                     // 위에 계산한 점수를 토탈 점수에 합산
    
    mvprintw(SCORE_MESSAGE_Y, 39, "SCORE: %6d", Total); // 해당 위치에 score점수를 보여줌
    
    /* 레벨과 현재 점수를 비교하여 레벨에 비해 점수가 높다면 블록이 내려오는 속도와 현재 레벨을
     현재 점수에 맞게 변경 */
    
    if((Select_no <= 1) && (Total>=LEVEL2_SCORE)){
        BlockSpeed = LEVEL2_SPEED;
        Level_no=2;
    }
    if((Select_no <= 2) && (Total>=LEVEL3_SCORE)){
        BlockSpeed = LEVEL3_SPEED;
        Level_no=3;
    }
    if((Select_no <= 3) && (Total>=LEVEL4_SCORE)){
        BlockSpeed = LEVEL4_SPEED;
        Level_no=4;
    }
    if((Select_no <= 4) && (Total>=LEVEL5_SCORE)){
        BlockSpeed = LEVEL5_SPEED;
        Level_no=5;
    }
    
    mvprintw(LEVEL_MESSAGE_Y, 39, "LEVEL %d",Level_no); // 현재 레벨이 몇인지 보여줌
}

/************************************************************************
 *                                    *
 *                Draw_Menu                *
 *                                    *
 ************************************************************************/
void Draw_Menu(void) {                  // 레벨 선택 메뉴 창을 출력해주는 함수
    
    int ch;                             // 값을 입력받을 변수
    int no=0;                           //
    int enter_flag=0;                   // 엔터 값을 저장하기 위한 변수 선언 및 0으로 초기화
    
    mvprintw(12,37,"Select Level");     // 메뉴의 내용을 출력 (레벨에 따라 속도가 다름)
    mvprintw(14,40,"LEVEL1");
    mvprintw(15,40,"LEVEL2");
    mvprintw(16,40,"LEVEL3");
    mvprintw(17,40,"LEVEL4");
    mvprintw(18,40,"LEVEL5");
    mvprintw(20,30,"(use arrow key and press ENTER)");  // 값을 지정 후, 엔터 키를 누르도록 안내
    
    // 선택 레벨이 1이라면 회색 배경색을 보여주고, 선택 레벨이 1이 아니라면 attroff를 통해 배경색을 끔(초기 메뉴에서 레벨 1로 자동 표시)
    (Select_no==1)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
    mvprintw(14,40,"LEVEL1");           // 레벨1을 표시
    
    while(ch=getch()){                  // 값을 입력받아 반복
        switch(ch) {
            case KEY_UP:                // UP KEY를 누를 경우 선택번호를 1 감소 후 빠져나옴
                Select_no--;
                break;
            case KEY_DOWN:              // DOWN KEY를 누를 경우 선택번호를 1 증가 후 빠져나옴
                Select_no++;
                break;
            default :                   // 그 외 키를 누를 경우 아무 작동 하지않음
                break;
        }
        
        
        // 초기 값을 1로 지정해놨기 때문에, 레벨 단계를 맞추기 위해 변경
        if(Select_no==6)
            Select_no=5;
        if(Select_no==0)
            Select_no=1;
        
        
        // 선택 레벨일 경우 배경 색 표시하고, 선택 레벨이 아닐 경우 배경 색 off 시킴
        (Select_no==1)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
        mvprintw(14,40,"LEVEL1");
        
        (Select_no==2)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
        mvprintw(15,40,"LEVEL2");
        
        (Select_no==3)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
        mvprintw(16,40,"LEVEL3");
        
        (Select_no==4)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
        mvprintw(17,40,"LEVEL4");
        
        (Select_no==5)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
        mvprintw(18,40,"LEVEL5");
        
        attroff(COLOR_PAIR(8));
        
        switch(Select_no) {                 // 선택한 레벨에 맞게 속도 조정
            case 1:
                BlockSpeed=LEVEL1_SPEED;
                break;
            case 2:
                BlockSpeed=LEVEL2_SPEED;
                break;
            case 3:
                BlockSpeed=LEVEL3_SPEED;
                break;
            case 4:
                BlockSpeed=LEVEL4_SPEED;
                break;
            case 5:
                BlockSpeed=LEVEL5_SPEED;
                break;
        }
        
        if(ch == '\n')                      // enter키를 입력 받았을 경우 빠져나옴
            break;
    }
    
    Level_no=Select_no;                     // 레벨에 선택한 레벨로 변경
    mvhline(12,30,' ',30);                  // 메뉴에 표시된 내용 지우기
    mvhline(14,30,' ',30);
    mvhline(15,30,' ',30);
    mvhline(16,30,' ',30);
    mvhline(17,30,' ',30);
    mvhline(18,20,' ',30);
    mvhline(19,30,' ',30);
    mvhline(20,30,' ',50);
    mvprintw(16,39,"PAUSE : press 'P'");    // 해당 위치에 PAUSE : press 'P' 출력
}

/************************************************************************
 *                                    *
 *                Draw_Menu2                *
 *                                    *
 ************************************************************************/
void Draw_Menu2(void)       // P를 눌렀을 경우 호출되는 함수
{
    int ch;                 // 입력받을 변수 선언
    int no=1;               // 선택할 값을 저장할 변수를 선언 및 1로 초기화
    int enter_flag=0;       // 엔터를 눌렀는지 안눌렀는지 확인하는 flag 선언
    
    mvprintw(18,39,"Resume Game        ");                 // 해당 내용 출력
    mvprintw(19,39,"Quit Game");
    mvprintw(22,35,"(use arrow key and press ENTER)");
    
    while(ch=getch()){       // 입력받은 값을 저장하면서 무한 루프
        switch(ch) {
            case KEY_UP:     // UP KEY를 눌렀을 경우 no에 1 감소 후, 빠져나옴
                --no;
                break;
            case KEY_DOWN:   // DOWN KEY를 눌렀을 경우 no에 1 증가 후, 빠져나옴
                ++no;
                break;
            case '\n':       // 엔터를 눌렀을 경우 enter_flag 1로 설정
                enter_flag=1;
        }
        
        // 초기값이 1로 설정되어 있기 때문에, 값을 맞추기 위해 변경
        if(no==0)
            no=1;
        if(no==3)
            no=2;
        
        // no가 선택되었을 경우 배경 색 표시, 선택되지 않을 경우 배경 색 off
        (no==1)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
        mvprintw(18,39,"Resume Game    ");
        
        (no==2)?attron(COLOR_PAIR(8)):attroff(COLOR_PAIR(8));
        mvprintw(19,39,"Quit Game");
        
        attrset(A_NORMAL);
        
        if(enter_flag==1) {
            if(no==2) {         // Quit Game을 선택했을 경우
                erase ();       // 화면 초기화
                refresh ();     // 버퍼 초기화
                endwin ();      // curses 모드 종료
                exit(0);        // 강제 종료
            }
            
            // Restart Game 선택 시, 테트리스가 다시 움직이게 만들음
            mvhline(14,30,' ',30);
            mvhline(15,30,' ',30);
            mvhline(16,30,' ',30);
            mvhline(17,30,' ',30);
            mvhline(18,30,' ',30);
            mvhline(19,30,' ',30);
            mvhline(22,30,' ',50);
            mvprintw(16,39,"PAUSE : press 'P'");    // 해당 위치에 PAUSE : press 'P' 출력
            break;                                  // 빠져나옴
        }
    }
}

/************************************************************************
 *                                    *
 *                start                    *
 *                                    *
 ************************************************************************/
void start(void)
{
    initscr ();                         // Curses 모드 초기화
    cbreak ();                          // 엔터를 누르지 않아도 입력이 전달됨
    noecho ();                          // 사용자로부터 입력받은 문자 에코 하지 않음
    keypad (stdscr, TRUE);              // F(평션)키 사용가능하도록 함 (방향키를 위해 사용)
    start_color ();                     // color 색상 사용
    curs_set(0);                        // 커서 표시 지움
    
    
    // pair 색상 설정
    
    init_pair (1, COLOR_CYAN,  COLOR_CYAN);
    init_pair (2, COLOR_RED,  COLOR_RED);
    init_pair (3, COLOR_WHITE,  COLOR_WHITE);
    init_pair (4, COLOR_YELLOW,  COLOR_YELLOW);
    init_pair (5, COLOR_GREEN,  COLOR_GREEN);
    init_pair (6, COLOR_MAGENTA,  COLOR_MAGENTA);
    init_pair (7, COLOR_BLUE,  COLOR_BLUE);
    init_pair (8, COLOR_BLACK,  COLOR_WHITE);
    
    srand((unsigned)time(NULL));        // srand 함수에 time 시드값 초기화하여 랜덤하게 사용
    
    Init_Game();                        // 게임을 시작하는 함수 호출
    Draw_Menu();                        // 시작 전, 레벨을 선택하는 메뉴를 보여주는 함수 호출
    Draw_Wall();                        // 게임이 시작되고 테두리를 그려주는 함수 호출
    Block_No2 = rand()%7;               // Block_No2을 랜덤하게 생성
    preview2();                         // NEXT Block을 미리 보여주는 함수 호출
    
    Block_No=rand()%7;                  // 게임 시작 첫 Block을 랜덤하게 생성
    
    for( ; ; ){                         // 무한 루프를 통해 반복
        Draw_Block();                   //
        wtimeout(stdscr, BlockSpeed);    // 내려오는 시간을 BlockSpeed를 통해 속도 조절
        if (0 != key_in())              // 키를 입력할 수 없으면 빠져나옴
            break;
        else if (0 != Down_Proc())      // Down_Proc할 수 없으면 빠져나옴
            break;
        Draw_Game();                    // 게임 종료
    }
    
    refresh ();                         // 버퍼 초기화
    endwin ();                          // curses 모드 종료
}

/************************************************************************
 *                                    *
 *                main                    *
 *                                    *
 ************************************************************************/
int main(void)
{
    start();
    return 0;
}
~~~

