# Write-up-HCMUS

## BabyDroid - 200đ 

**ủa, bài này k biết sao debug bị lỗi nên full static analysis =)))**


Đầu tiên đề cho ta 1 file apk chỉ dùng để check flag cơ bản

![image](https://user-images.githubusercontent.com/88520787/168464204-2cc2b133-f334-4e78-b02b-9784407bd584.png)

Mình đã dùng trang [Android apk decompiler](http://www.javadecompilers.com/apk) để decompile file để đọc source

Down source về ta tìm trong đường dẫn `\sources\com\example\babydroid` sẽ có các file java của chương trình gốc.

![image](https://user-images.githubusercontent.com/88520787/168463840-ef2fd218-d428-4d75-b00a-f5cb8f6df63d.png)

Đọc thử file `FlagValidator.java` (file dùng để check flag) ta thấy có 1 hàm check flag với 1 loạt các điều kiên:

```java
package com.example.babydroid;

import android.content.Context;

public class FlagValidator {

    public static boolean checkFlag(Context ctx, String flag) {
        String regex = retriever();
        if (flag.startsWith("HCMUS-CTF{") &&
         flag.charAt(19) == '_' &&
        flag.length() == 37 &&
        flag.toLowerCase().substring(10).startsWith("this_is_") &&
         flag.charAt(((int) (((double) MagicNum.obtainY()) * Math.pow((double) MagicNum.obtainX(), (double) MagicNum.obtainY()))) + 2) == flag.charAt(((int) Math.pow(Math.pow(2.0d, 2.0d), 2.0d)) + 3) &&
        new StringBuilder(flag).reverse().toString().toLowerCase().substring(1).startsWith(ctx.getString(C0095R.string.last_part)) &&
        new StringBuilder(flag).reverse().toString().charAt(0) == '}' &&
        Helper.ran(flag.toUpperCase().substring((MagicNum.obtainY() * MagicNum.obtainX() * MagicNum.obtainY()) + 2, (int) (Math.pow((double) MagicNum.obtainZ(), (double) MagicNum.obtainX()) + 1.0d))).equals("ERNYYL") &&
        flag.toLowerCase().charAt(18) == 'a' &&
        flag.charAt(18) == flag.charAt(28) &&
        flag.toUpperCase().charAt(27) == flag.toUpperCase().charAt(28) + 1) {
            return flag.substring(10, flag.length() - 1).matches(regex);
        }
        return false;
    }
}
```

Kiểm tra hết các mệnh đề cơ bản, ta được 1 phần của flag và biết được len(flag) = 37 kí tự:

`HCMUS-CTF{this_is_a_*******ba*******}`

Và hiện giờ ta chỉ cần check thêm 3 điều kiện:

```java
flag.charAt(((int) (((double) MagicNum.obtainY()) * Math.pow((double) MagicNum.obtainX(), (double) MagicNum.obtainY()))) + 2) == flag.charAt(((int) Math.pow(Math.pow(2.0d, 2.0d), 2.0d)) + 3) &&
        new StringBuilder(flag).reverse().toString().toLowerCase().substring(1).startsWith(ctx.getString(C0095R.string.last_part)) &&
        Helper.ran(flag.toUpperCase().substring((MagicNum.obtainY() * MagicNum.obtainX() * MagicNum.obtainY()) + 2, (int) (Math.pow((double) MagicNum.obtainZ(), (double) MagicNum.obtainX()) + 1.0d))).equals("ERNYYL")
```
        
để biết được `MagicNum.obtain` mình đã kiểm tra trong file `MagicNum.java`:

```java
public static int obtainX() {
        return 2;
    }

    public static int obtainY() {
        return 3;
    }

    public static int obtainZ() {
        return 5;
    }
```
Thay các số tương ứng, ta được các điều kiện ngắn hơn:

check điều kiện đầu tiên:

```java
flag.charAt(((int) (((double) 3) * Math.pow((double) 2, (double) 3))) + 2) == flag.charAt(((int) Math.pow(Math.pow(2.0d, 2.0d), 2.0d)) + 3)
```
đoạn này chạy các code bên trong sẽ ra 1 con số cụ thể, sau đó, đoạn này sẽ có nghĩa là flag[26]==flag[19] (kí tự '_')

=> Flag hiện tại ``HCMUS-CTF{this_is_a_******_ba*******}`
```java
Helper.ran(flag.toUpperCase().substring((3 * 2 * 3) + 2, (int) (Math.pow((double) 5, (double) 2) + 1.0d))).equals("ERNYYL")
```
Đoạn này sẽ thay thế vào chổ thiếu 6 kí tự, kiểm tra hàm `ran()` trong file `Helper.java`:
```java
public static String ran(String s) {
        String out = "";
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c >= 'a' && c <= 'm') {
                c = (char) (c + 13);
            } else if (c >= 'A' && c <= 'M') {
                c = (char) (c + 13);
            } else if (c >= 'n' && c <= 'z') {
                c = (char) (c - 13);
            } else if (c >= 'N' && c <= 'Z') {
                c = (char) (c - 13);
            }
            out = out + c;
        }
        return out;
    }
```
Hàm này chỉ thay đổi từng kí tự cơ bản, dùng nó với chuỗi đề cho `"ERNYYK"`, ta được chuỗi `"REALLY"`

Flag `HCMUS-CTF{this_is_a_REALLY_ba*******}` chỉ cần tìm phần cuối thông qua:

```java
new StringBuilder(flag).reverse().toString().toLowerCase().substring(1).startsWith(ctx.getString(C0095R.string.last_part))
```

tạm thời ta chỉ quan tâm tới `ctx.getString(C0095R.string.last_part)`, kiểm tra trong file `C0095R.java`: ta có được id của last_part:
```java
public static final int last_part = 2131623979;
```
Qua tìm hiểu, thì `context.getString(<id>)` sẽ trả về chuỗi của trương trình với `id` tương ứng;

ta tìm các string của chương trình nằm ở `\resources\res\values\strings.xml`:

![image](https://user-images.githubusercontent.com/88520787/168465618-277e4961-d0ee-478b-b45e-cc3c7ac74b05.png)

Đảo ngược chuỗi và ghép vào flag, ta được:

`HCMUS-CTF{this_is_a_REALLY_basic_rev}`

### Còn bước nữa

Sau khi pass toàn bộ điều kiện của `if`, hàm `checkFlag` còn có đoạn kiểm tra flag người dùng với `regex` mà chương trình có thông qua hàm `retriever()` nằm trong file `Helper.java`:

```java
public static String retriever() {
        StringBuilder sb;
        String str;
        String r = "";
        boolean upper = true;
        for (int i = 0; i < 26; i++) {
            if (upper) {
                sb = new StringBuilder();
                sb.append(r);
                str = "[A-Z_]";
            } else {
                sb = new StringBuilder();
                sb.append(r);
                str = "[a-z_]";
            }
            sb.append(str);
            r = sb.toString();
            upper = !upper;
        }
        return r;
}
```
Chạy đoạn này, ta được regex: `[A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_][A-Z_][a-z_]`

Để flag match được với đoạn regex này thì flag sẽ có kí tự in thường và in hoa xen kẽ:

![image](https://user-images.githubusercontent.com/88520787/168465904-4c7fd4ea-1bfd-417b-8dff-5f13e05ad800.png)

![image](https://user-images.githubusercontent.com/88520787/168465918-8ef8ac68-3074-43ad-86f2-56059c5a2737.png)

Flag:
```
HCMUS-CTF{ThIs_iS_A_ReAlLy_bAsIc_rEv}
```

   
