## 记录遇到的一些好玩的算法

### 1.一种很有意思的反转数字的方法

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

### 2.实现destructuringArray方法，达到如下效果
```
// destructuringArray( [1,[2,4],3], "[a,[b],c]" );
// result
// { a:1, b:2, c:3 }
```
自己的实现思路就是用递归对数组进行操作，具体的代码见下
```
const destructuringArray = (arr, str) => {
    const res = {}
    const switchArr = JSON.parse(str.replace(/\w+/g, match => `"${match}"`))
    const fn =  (arr, switchArr) => {
        switchArr.map((item, index) => {
            if(Object.prototype.toString.call(item) === '[object Array]') {
                fn(arr[index], item)
            } else {
                res[item] = arr[index]
            }
        })
    }
    fn(arr, switchArr)
    return res
}
destructuringArray([1,[2, 4],3], "[a,[b],c]")
```
