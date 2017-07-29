## 记录遇到的一些好玩的算法

### 一种很有意思的反转数字的方法

对于一个整数X，定义操作rev(X)为将X按数位翻转过来，并且去除掉前导0。例如:

如果 X = 123，则rev(X) = 321;

如果 X = 100，则rev(X) = 1.

现在给出整数x和y,要求rev(rev(x) + rev(y))为多少？

```
import java.util.Scanner;
public class Main{
    public static void main(String[] args){
        Scanner in = new Scanner(System.in);
        int x = in.nextInt();
        int y = in.nextInt();
        int result = rev(rev(x)+rev(y));
        System.out.print(result);
    }
    
	public static int rev(int num) {
	    int t = 0;
	    while(num!=0) {
	        t = t*10+num%10;
	        num = num/10;
	    }
	    return t;
	}
} 
```