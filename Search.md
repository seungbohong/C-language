- # Search



1.  보초법을 사용한 선형검색 소스코드

~~~ c
#include <stdio.h>
#include <stdlib.h>

int search(int a[], int n, int key){
    int i  = 0;
    a[n] = key; // 보초법을 사용
    while(1){
        if(a[i] == key)
            break;
        i++;
    }
    return i == n ? -1 : i;
}

int main() {
    // insert code here...
    int i, nx, ky, idx;
    int *x;
    puts("선형 검색");
    printf("요소 개수 : ");
    scanf("%d", &nx);
    x = calloc(nx+1, sizeof(int));  //요소의 개수가 nx개인 int형 배열을 생성
    for(i = 0; i < nx; i++){
        printf("x[%d] : ", i);
        scanf("%d", &x[i]);
    }
    printf("검색값 : ");
    scanf("%d", &ky);
    idx = search(x, nx, ky);
    if(idx == -1)
        puts("failed to search.");
    else
        printf("%d(은) x[%d]에 있었습니다.\n", ky, idx);
    free(x);
    
    
    
    return 0;
}

~~~



2. 정렬되있는 요소들을 검색하는 이진 탐색

~~~ c
#include <stdio.h>
#include <stdlib.h>

// 요소의 개수가 n인 배열 a에서 key와 일치하는 요소를 이진 탐색
int bin_search(const int a[], int n, int key){
    int pl = 0;
    int pr = n-1;
    int pc;
    do{
        pc = (pl+pr)/2;
        if(a[pc] == key)
            return pc;
        else if(a[pc] < key)
            pl = pc+1;
        else
            pr = pc-1;
    }while(pl <= pr);
    return -1;
    
}

int main() {
    // insert code here...
    int i, nx, ky, idx;
    int *x;
    puts("이진 검색");
    printf("요소 개수 : ");
    scanf("%d", &nx);
    x = calloc(nx, sizeof(int));
    printf("오름차순으로 입력하세요.\n");
    printf("x[0] : ");
    scanf("%d", &x[0]);
    for(i=1; i<nx; i++){
        do{
            printf("x[%d] : ", i);
            scanf("%d", &x[i]);
        }while(x[i] < x[i-1]);  //바로 앞의 값보다 작으면 다시 입력
        }
    printf("검색값 : ");
    scanf("%d", &ky);
    idx = bin_search(x, nx, ky);
    if(idx == -1)
        puts("검색에 실패했습니다.");
    else
        printf("%d는 x[%d]에 있습니다.\n", ky, idx);
    free(x);
    
    return 0;
}
~~~



3. bsearch(정렬된 배열에서 검색하는 함수)

~~~c
#include <stdio.h>
#include <stdlib.h>

int int_cmp(const int *a, const int *b)
{
    if(*a < *b)
        return -1;
    else if(*a > *b)
        return 1;
    else
        return 0;
}

int main() {
    // insert code here...
    int i, nx, ky;
    int *x;              //배열의 첫번째 요소에 대한 포인터
    int *p;              //검색한 요소에 대한 포인터
    puts("bsearch 함수를 사용하여 검색");
    printf("요소 개수 : ");
    scanf("%d", &nx);
    x = calloc(nx, sizeof(int));
    
    printf("오름 차순으로 입력하시오\n");
    printf("x[0] : ");
    scanf("%d", &x[0]);
    for(i=1; i<nx; i++){
        do{
            printf("x[%d] : ", i);
            scanf("%d", &x[i]);
        }while(x[i] < x[i-1]);
    }
    printf("검색값 : ");
    scanf("%d", &ky);
    p = bsearch(&ky, x, nx, sizeof(int), (int(*)(const void *, const void *))int_cmp);
    //p의 값에 key값의 주소, 배열의 포인터, 갯수, 크기, 비교함수(형변환)
    
    if(p == NULL)
        puts("검색에 실패했습니다.");
    else
        printf("%d는 x[%d]에 있습니다.\n", ky, (int)(p-x));
    free(x);
    
    
    return 0;
}
~~~

