
# Khái niệm về Privilege Escalation

- Trong 1 hệ điều hành nào đó, các file, app, tệp được phân quyền 1 cách rõ ràng. 1 user nào đó có thể có các quyền như:
	- Đọc
	- Viết
	- Thực thi
- 1 User được định nghĩa là có quyền cao hơn nếu có nhiều quyền hơn trong các tệp các file của hệ thống
- 1 Hacker muốn đọc 1 file, đi vào đường dẫn blah blah nào đó đều phải có quyền tương ứng
- Privilege Escalation là một trường phái mà bạn dựa vào các SUID, các tiến trình mà High Privilege user, hoặc nôm na là 1 config lỗi của hệ điều hành và kernel sử dụng. Từ đó bạn có thể lợi dụng các cái lỗ hổng đó mà lấy được shell của đối tượng mà bạn muốn coi như là bạn đã chiếm quyền 1 user
- Việc chiếm quyền rất quan trọng khi mà chiếm quyền 1 server nào đó thì bạn thường xuất phát điểm ở user có low privilege nhất.

# Các hệ điều hành sẽ học

## [[Windows]]



## [[Linux]]

## MacOS thì cút mẹ đi giống Linux cả thôi


