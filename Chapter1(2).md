# Chapter1(2)

- Word Counting

~~~ c
#include <stdio.h>

#define IN 1
#define OUT 0

int main(){
    int c, nl, nw, nc, state;
    
    state = OUT;
    nl = nw = nc = 0;
    while((c=getchar()) != EOF){
        ++nc;
        if(c == '\n')
        	++nl;
        if(c == ' ' || c == '\n' || c == '\t')
        	state = OUT;
        else if (state == OUT){
            state = IN;
            nw++;
        }
    }
    printf("%d %d %d\n", nl, nw, nc);
    return 0;
}
~~~



- Arrays

~~~ c
#include <stdio.h>

/*문자를 하나씩 읽어 char형 숫자가 총 몇개고 공백문자는 몇개고 그 외의 문자는 몇개인지 출력해주는 함수*/
int main(){
    int c, i, nwhite, nother;
    int ndigit[10];
    
    nwhite = nother = 0; //nwhite는 공백개수, nother는 숫자이외의 문자열
    for(i=0; i<10; i++)
        ndigit[i] = 0;
    
    while((c = getchar()) != EOF){
        if (c >= '0' && c <= '9')
            ++ndigit[c-'0'];
        else if (c == ' ' || c == '\n' || c == '\t')
            ++nwhite;
        else
            ++nother;
    }
    
    printf("digits = ");
    for(i=0; i < 10; i++)
        printf(" %d", ndigit[i]);
    printf(", whitespace = %d, other = %d\n", nwhite, nother);
    
    return 0;
}
~~~



- Functions

~~~ c
#include <stdio.h>

int power(int m, int n);

void main(){
    int i;
    
    for(i=0; i<10; ++i)
    	printf("%d %d %d\n", i, power(2, i), power(-3, i));
    	return 0;
}

int power(int base, int n){
    int i, p;
    
    p=1;
    for(i=1; i<=n; ++i)
    	p = p*base;
    return p;
}
~~~



- Character Arrays

  getline과 copy를 사용해서 문자열을 복사하고 출력함

  **현재 입력값이 최대길이의 문자열이면 문자를 가져와 복사후 출력해주는 프로그램**


~~~ c
#include <stdio.h>
#define MAXLINE 1000

int get_line(char line[], int maxline);
void copy(char to[], char from[]);

int main(){
    int len;             		// current line length
    int max;			 		// maximum length seen so far
    char line[MAXLINE];	 		// current input line
    char longest[MAXLINE];  	// longest line saved here
    
    max = 0;
    while ((len = get_line(line, MAXLINE)) > 0)
        if (len > max){
            max = len;
            copy(longest, line);
        }
    if (max > 0)
        printf("%s", longest);
    return 0;
}

int get_line(char s[], int lim){
    int c, i;
    
    for(i=0; i<lim-1 && (c=getchar())!=EOF && c != '\n'; ++i)
        s[i] = c;
    if(c == '\n'){
        s[i] = c;
        ++i;
    }
    s[i] = '\0';
    return i;
}

void copy(char to[], char from[]){
    int i;
    
    i=0;
    while((to[i] = from[i]) != '\0')
        ++i;
}
~~~

