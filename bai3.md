# BÀI 3: Đọc hiểu & Dò lỗi qua Prompt (Phát hiện lỗi logic lặp)

## 1. Phân tích vì sao prompt thô **"Mã này bị lỗi gì?"** dễ khiến AI bỏ sót lỗi logic

Prompt **"Mã này bị lỗi gì?"** là một prompt quá ngắn và quá chung chung, nên rất dễ khiến AI bỏ sót lỗi logic trong đoạn code `DuplicateFinder`. Lý do là vì lỗi ở đây **không phải lỗi cú pháp (syntax error)**, cũng **không phải lỗi biên dịch (compile error)**, mà là **lỗi logic chạy sai kết quả** trong một trường hợp kiểm thử cụ thể.

---

# 2. Vì sao prompt thô dễ bị bỏ sót lỗi?

## 2.1. Code hoàn toàn hợp lệ về mặt cú pháp và compile

Đoạn code sau:

```java
public class DuplicateFinder {

    public static Integer findDuplicate(int[] arr) {

        for (int i = 0; i < arr.length; i++) {

            for (int j = i; j < arr.length; j++) {

                if (arr[i] == arr[j]) {

                    return arr[i];

                }

            }

        }

        return null;

    }

}
```

**không hề có lỗi cú pháp**:

* khai báo class đúng;
* vòng lặp `for` đúng;
* kiểu trả về `Integer` đúng;
* không có lỗi biên dịch.

Vì vậy, nếu chỉ hỏi AI bằng câu chung chung như **“Mã này bị lỗi gì?”**, AI có thể chỉ dừng ở mức rà soát:

* có lỗi cú pháp không,
* có lỗi compile không,
* có warning rõ ràng nào không.

Trong khi lỗi thật sự lại nằm ở **hành vi logic khi chạy**.

---

## 2.2. Prompt không cung cấp ca kiểm thử để phơi bày lỗi

Lỗi của đoạn code nằm ở vòng lặp trong:

```java
for (int j = i; j < arr.length; j++)
```

Do `j` bắt đầu từ `i`, nên ngay lần lặp đầu tiên của vòng trong, ta luôn có:

```java
arr[i] == arr[j]  // tức là arr[i] == arr[i]
```

Biểu thức này **luôn đúng**, nên hàm sẽ trả về ngay `arr[i]` ở lần kiểm tra đầu tiên.
Kết quả là hàm **luôn trả về phần tử đầu tiên của mảng** thay vì “phần tử trùng lặp đầu tiên” như mong muốn.

Ví dụ với mảng:

```java
{1, 2, 3, 4}
```

mảng này **không có phần tử trùng**, nhưng hàm vẫn trả về `1` vì:

* `i = 0`
* `j = 0`
* `arr[0] == arr[0]` → đúng
* hàm trả về `arr[0] = 1`

Nếu prompt không đưa ra ca kiểm thử như trên, AI rất dễ **không phát hiện được lỗi hành vi** vì code nhìn qua có vẻ “đúng cấu trúc”.

---

## 2.3. Prompt không chỉ rõ cần phân tích lỗi logic biên

Câu hỏi **“Mã này bị lỗi gì?”** không nói rõ AI phải:

* kiểm tra **logic thuật toán**;
* chạy thử bằng **test case cụ thể**;
* tìm **trường hợp phản ví dụ**;
* phân tích **đầu vào không có phần tử trùng**.

Vì thiếu định hướng đó, AI có thể:

* trả lời quá hời hợt,
* chỉ nói “đây là hàm tìm phần tử trùng lặp” mà không phát hiện bug,
* hoặc chỉ gợi ý sửa nhỏ như “kiểm tra null” mà bỏ qua lỗi cốt lõi.

---

## 2.4. Prompt không yêu cầu tối ưu thuật toán

Ngoài lỗi logic, đề bài còn muốn **nâng cấp cách giải từ O(N²) xuống O(N)** bằng `HashSet`.
Prompt thô không hề yêu cầu điều này, nên ngay cả khi AI phát hiện bug, nó vẫn có thể sửa theo hướng rất hẹp như:

```java
for (int j = i + 1; j < arr.length; j++)
```

Cách sửa đó giúp **đúng logic hơn**, nhưng vẫn giữ độ phức tạp **O(N²)**, chưa đạt yêu cầu tối ưu của đề bài.

---

# 3. Kết luận về nhược điểm của prompt thô

Prompt **“Mã này bị lỗi gì?”** dễ khiến AI bỏ sót lỗi vì nó thiếu gần như toàn bộ các thành phần quan trọng của một prompt kỹ thuật tốt:

* **Thiếu vai trò**: không yêu cầu AI đóng vai Code Auditor / Debugger.
* **Thiếu ngữ cảnh**: không nêu rõ mục tiêu của hàm là “tìm phần tử trùng lặp đầu tiên”.
* **Thiếu ca kiểm thử phản ví dụ**: không đưa mảng `{1, 2, 3, 4}` để lộ lỗi.
* **Thiếu ràng buộc**: không yêu cầu phân tích root cause và tối ưu hiệu năng.
* **Thiếu định dạng đầu ra**: không yêu cầu AI trả về mã nguồn Java đã sửa hoàn chỉnh.

Vì vậy, để AI phát hiện đúng lỗi và sửa đúng hướng, cần thiết kế một prompt mới **cụ thể hơn, có test case, có vai trò, có mục tiêu và có ràng buộc kỹ thuật rõ ràng**.

---

# 4. Prompt tối ưu mới thiết kế

````text
Bạn là một **Code Auditor** chuyên rà soát lỗi logic và tối ưu thuật toán trong mã nguồn Java.

## Bối cảnh
Tôi có một hàm Java dùng để tìm **phần tử trùng lặp đầu tiên** trong một mảng số nguyên. Tuy nhiên, hàm hiện tại đang bị lỗi logic: nó có thể trả về sai kết quả ngay cả khi mảng **không có phần tử trùng**.

## Mã nguồn cần phân tích
```java
public class DuplicateFinder {

    // Lỗi logic: j bắt đầu từ i dẫn đến việc phần tử tự so sánh với chính nó (arr[i] == arr[i] luôn đúng)

    public static Integer findDuplicate(int[] arr) {

        for (int i = 0; i < arr.length; i++) {

            for (int j = i; j < arr.length; j++) {

                if (arr[i] == arr[j]) {

                    return arr[i];

                }

            }

        }

        return null;

    }

}
````

## Ca kiểm thử bắt buộc phải phân tích

Hãy kiểm tra hành vi của hàm với input sau:

```java
int[] arr = {1, 2, 3, 4};
```

Đây là mảng **không có phần tử trùng**, nhưng code hiện tại vẫn trả về `1`.
Bạn hãy giải thích **vì sao điều đó xảy ra**, chỉ ra **root cause** trong vòng lặp lồng nhau, và phân tích lỗi logic thật rõ ràng.

## Nhiệm vụ

1. Phân tích lỗi logic của đoạn code hiện tại.
2. Giải thích vì sao với mảng `{1, 2, 3, 4}` hàm lại trả về sai kết quả.
3. Không chỉ sửa lỗi bằng cách đổi `j = i + 1`, mà hãy **viết lại giải pháp tốt hơn bằng `HashSet`** để:

   * tìm phần tử trùng lặp đầu tiên đúng nghĩa,
   * giảm độ phức tạp thời gian từ **O(N²)** xuống **O(N)**.
4. Xử lý thêm các trường hợp biên:

   * mảng `null`
   * mảng rỗng
5. Giữ nguyên kiểu trả về:

   ```java
   Integer
   ```

## Định dạng đầu ra

1. Phần 1: Phân tích lỗi logic và root cause.
2. Phần 2: Giải thích vì sao test case `{1, 2, 3, 4}` bị trả sai.
3. Phần 3: Mã nguồn Java hoàn chỉnh đã sửa bằng `HashSet` trong khối code Markdown.
4. Phần 4: So sánh độ phức tạp thời gian trước và sau khi tối ưu.

````

---

# 5. Mã nguồn Java đã sửa hoàn chỉnh bằng HashSet

Dưới đây là phiên bản đã sửa đúng logic và tối ưu hơn bằng `HashSet`.

```java
import java.util.HashSet;
import java.util.Set;

public class DuplicateFinder {

    public static Integer findDuplicate(int[] arr) {
        if (arr == null || arr.length == 0) {
            return null;
        }

        Set<Integer> seen = new HashSet<>();

        for (int num : arr) {
            if (seen.contains(num)) {
                return num;
            }
            seen.add(num);
        }

        return null;
    }
}
````

---

# 6. Giải thích cách sửa

## 6.1. Ý tưởng của phiên bản mới

Thay vì dùng 2 vòng lặp lồng nhau để so sánh từng cặp phần tử, ta dùng `HashSet` để ghi nhớ các phần tử đã gặp.

Quy trình:

1. Duyệt từng phần tử trong mảng từ trái sang phải.
2. Nếu phần tử hiện tại **đã tồn tại trong `HashSet`**, nghĩa là đây là phần tử trùng lặp đầu tiên → trả về ngay.
3. Nếu chưa tồn tại, thêm phần tử đó vào `HashSet`.
4. Nếu duyệt hết mảng mà không có phần tử trùng → trả về `null`.

---

## 6.2. Ví dụ hoạt động

### Trường hợp 1: Có phần tử trùng

```java
int[] arr = {4, 2, 7, 2, 9};
```

Diễn biến:

* gặp `4` → thêm vào set
* gặp `2` → thêm vào set
* gặp `7` → thêm vào set
* gặp `2` lần nữa → đã tồn tại trong set → trả về `2`

Kết quả:

```java
2
```

---

### Trường hợp 2: Không có phần tử trùng

```java
int[] arr = {1, 2, 3, 4};
```

Diễn biến:

* `1` chưa có → thêm
* `2` chưa có → thêm
* `3` chưa có → thêm
* `4` chưa có → thêm

Duyệt xong không có phần tử nào lặp lại → trả về:

```java
null
```

Đây mới là hành vi đúng với yêu cầu bài toán.

---

# 7. So sánh độ phức tạp trước và sau khi sửa

## Phiên bản cũ

Phiên bản ban đầu dùng 2 vòng lặp lồng nhau:

```java
for (int i = 0; i < arr.length; i++) {
    for (int j = i; j < arr.length; j++) {
        ...
    }
}
```

Độ phức tạp thời gian là:

```text
O(N²)
```

---

## Phiên bản mới dùng HashSet

Phiên bản mới chỉ duyệt mảng **1 lần**, mỗi thao tác `contains()` và `add()` trên `HashSet` trung bình là **O(1)**.

Tổng độ phức tạp thời gian:

```text
O(N)
```

Độ phức tạp bộ nhớ phụ:

```text
O(N)
```

---

# 8. Kết luận cuối cùng

Prompt thô **“Mã này bị lỗi gì?”** rất dễ khiến AI bỏ sót lỗi vì:

* code không sai cú pháp,
* không có compile error,
* lỗi nằm ở **logic chạy sai trong một test case cụ thể**,
* prompt không đưa phản ví dụ để AI kiểm chứng hành vi.

Prompt tối ưu mới cần:

* đặt **vai trò rõ ràng** cho AI là **Code Auditor**;
* cung cấp **đầy đủ mã nguồn**;
* đưa **ca kiểm thử cụ thể** như `{1, 2, 3, 4}` để phơi bày lỗi;
* yêu cầu AI **phân tích root cause**;
* và buộc AI **viết lại lời giải bằng `HashSet`** để vừa sửa logic vừa tối ưu hiệu năng từ **O(N²)** xuống **O(N)**.

=> Vì vậy, cách tiếp cận đúng không chỉ là “hỏi AI code này lỗi gì”, mà là **thiết kế prompt có ngữ cảnh, có test case, có ràng buộc kỹ thuật và có đầu ra mong muốn rõ ràng**.
