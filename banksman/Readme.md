![image](https://github.com/user-attachments/assets/f25e9c79-86c3-4272-822c-9d7dfe5f0b12)


# Analyzing

Phân tích đề bài, mình nhận ra rằng đề bài có hint nhỏ là hãy tìm gì đó bí ẩn ở bên trong file pdf. 

## Step into

Ở bước đầu này, vì đây là một bài pdf, nên mình sẽ tiếp cận nó với 2 bước, bước đầu là kiểm tra xem đây có đúng là 1 file pdf và bước tiếp theo sẽ thử strings ra

![image](https://github.com/user-attachments/assets/5be04663-d46d-4fe2-a000-5574f1b0abd3)

Đây là 1 file pdf, vậy thì đến bước tiếp theo

![image](https://github.com/user-attachments/assets/bd0fd45d-d6f8-438e-b76a-e46164e072b5)

Vậy là trong file pdf có nhúng script JS. 1 file PDF nếu được nhúng các Actions Script như JS thì sẽ giúp cho file PDF có các tính năng như xác thực dữ liệu, tạo UI cho người dùng có thể tương tác. Nó sẽ được định nghĩa trong PDF bởi nametag /Action, ở đây thì script được nhúng là JS với nametag là /JavaScript

Rõ ràng nó có gì đó ba chấm ở đây, vì vậy nên mình sẽ dump toàn bộ chỗ Base64 encrypt của JS này ra 1 file để tiến hành Decode

## Decoding part

![image](https://github.com/user-attachments/assets/c1e21c64-e7b0-40a5-addf-1e2335f5bf92)

Giải thích sơ qua thì đây là 1 đoạn script có 1 func là **base64ToStream()** và 1 biến var **serialized_obj**

Giải thích về func **base64ToStream()**:

  Object ActiveXObject được gán với biến enc (encrypt), trong khi Object giúp cho JS có thể tạo và tương tác với COM (Component Object Model), nó cho phép JS có thể tương tác với các ứng dụng trên Windows và ta thấy rằng, nó đang gọi một lớp là System.Text.ASCIIEncoding của .NET, nó giúp encode Unicode thành dạng ASCII

  Tiếp theo tới biến length, và ba, đây đơn giản là các biến được gán với giá trị tham số được truyền vào của hàm base64ToStream là b, 2 biến này giúp tính byte và length của biến enc.

  Sau đó tới biến transform, cũng như biến enc, biến transform lúc này gọi class System.Security của .NET là FromBase64Transform, điều này có thể liên quan tới việc giúp chuyển hóa Base64 sang dữ liệu gốc hoặc gì đó để có thể chạy chương trình được ẩn bên trong dãy base.

  Và cuối cùng là MemoryStream, nó giúp lưu dữ liệu đã được decrypt từ biến transform.

Vậy là, mấu chốt vấn đề nằm ở đoạn base64 đó, bởi vì đoạn script này cũng chỉ dùng để encode và decode của 1 chương trình .NET. Vậy nên ta dump Base64 để tiến hành decode.


```
import re, base64

# Read the JavaScript file (adjust the filename as needed)
with open("E:\\CTF\\ritsec\\for\\banksman\\lmao.js", "r") as f:
    js_code = f.read()

# Use a regex to find all string literals containing Base64 data
# (assuming they are enclosed in double quotes)
matches = re.findall(r'"([A-Za-z0-9+/=]+)"', js_code)

# Concatenate the parts into one long string
combined_base64 = "".join(matches)

# Decode the combined Base64 data
decoded_data = base64.b64decode(combined_base64)

# Write the decoded binary data to a file for further analysis
with open("payload.bin", "wb") as f:
    f.write(decoded_data)

print("Payload dumped to payload.bin")

```

Và sau đó thành quả 

![image](https://github.com/user-attachments/assets/be641401-3418-463f-a6da-b544d4576440)

Cho vào DiE ta sẽ thấy rằng chương trình được viết là VB.NET aka Visual Basic cho .NET

![image](https://github.com/user-attachments/assets/dda9d631-aab2-4c65-ab18-a0b6db7858fa)

Tuy nhiên, bởi vì (có thể do mình dump ngu) file không thể detect được đây là định dạng PE, vì vậy nên ta không thể analyze bằng các công cụ như dnSpy hay dotPeek được. Lúc này mình thử kiểm tra file này bằng notepad để đọc bên trong có gì, thì mình thấy

![image](https://github.com/user-attachments/assets/1c008f79-36ac-4317-b9d4-103d948e9674)

# Flag: ```MetaCTF{I_4m_n0t_@_m1n3r_1_@m_a_b4nk5m4n}```
