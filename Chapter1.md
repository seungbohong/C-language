# Chapter1 : A Tutorial Introduction



- Fahrenheit - Celsius temperatures

  ~~~ c
  #include <stdio.h>
  
  int main(){
      int fahr, celsius;
      int lower, upper, step;
      
      lower = 0;        // lower limit of temperatures table
      upper = 300;	  // uppler limit
      step = 20; 		  // step size
      
      fahr = lower;
      while(fahr <= upper){
          celsius = 5 * (fahr-32)/ 9;
          printf("%d\t #d\n", fahr, celsius);
          fahr = fahr + step;
      }
      return 0;
  }
  ~~~



- floating-point arithmetic version

  ``` c
  #include <stdio.h>
  
  
  int main(){
      float fahr, celsius;  //float version of fahr, celsius
      int lower, upper, step;
      
      lower = 0;        // lower limit of temperatures table
      upper = 300;	  // uppler limit
      step = 20; 		  // step size
      
      fahr = lower;
      while(fahr <= upper){
          celsius = (5.0/9.0) * (fahr-32);
          printf("%3.0f #6.1f\n", fahr, celsius);
          fahr = fahr + step;
      }
      return 0;
  }
  ```



- For statements

  ``` c
  #include <stdio.h>
  
  int main()
  {
      int fahr;
      
      for(fahr = 0; fahr <= 300; fahr = fahr +20)
      	printf("%3d %6.1f\n", fahr, (5.0/9.0)*(fahr-32));
      return 0;
  }
  
  ```



-  Symbolic Constants

``` c
#include <stdio.h>

#define LOWER 0
#define UPPER 300
#define STEP 20
  
  int main()
  {
      int fahr;
      
      for(fahr = LOWER; fahr <= UPPER; fahr = fahr + STEP)
      	printf("%3d %6.1f\n", fahr, (5.0/9.0)*(fahr-32));
      return 0;
  }
  
```



- File Copying

```c
#include <stdio.h>

int main(){
    int c;
    
    c = getchar();
    while(c != EOF){
        putchar(c);
        c = getchar();
    }
    return 0;
}
```



- Character Counting

``` c
#include <stdio.h>

int main(){
    long nc;
    
    nc = 0;
    while(c != EOF)
        ++nc;
    printf("ld\n", nc);
    return 0;
}
```



- Line Counting

``` c
#include <stdio.h>

int main(){
    int c, nl;
    
    nl = 0;
    while((c=getchar()) != EOF)
    	if (c == '\n')
    		++nl;
    printf("d\n", nl);
    return 0;
}
```

