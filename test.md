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
  }
  ~~~

  [어미새의 블로그](https://steemit.com/@yahweh87)
