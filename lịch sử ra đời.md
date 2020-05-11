# I. Lịch sử ra đời docker
## 1. Vitualization
 Từ xưa, mô hình máy chủ thường là máy chủ vật lý + hệ điều hành (OS) + application.
 <img src ="https://i.imgur.com/OCNuS4g.png">

 Mô hình gặp phải vấn đề là vì một máy chủ chỉ cài được một OS nên gây ra việc không tận dụng hết tài nguyên, gây lãng phí. Đây là lý do công nghệ ảo hóa Vitualization ra đời.
 
 <img src = "https://i.imgur.com/73esvQW.png">

 Tuy nhiên, công nghệ Vitualization cũng nảy sinh một vài vấn đề như:
 - Về tài nguyên: Khi bạn chạy máy ảo, bạn phải cung cấp "cứng" dung lượng ổ cứng cũng như ram cho máy ảo đó, bật máy ảo lên để đó không làm gì thì máy thật cũng phải phân phát tài nguyên. 
 *Ví dụ khi tạo một máy ảo ram 2GB trên máy thật ram 4GB, lúc này máy thật sẽ mất 2GB ram cho máy ảo, kể cả khi máy ảo không dùng hết 2GB ram, đó là một sự lãng phí*
 - Về thời gian: Việc khởi động, shutdown khá lâu, có thể lên tới hàng phút.
 
 Do đó, người ta sinh ra công nghệ Containerlization.
 
 ## 2. Containerlization

 <img src="https://i.imgur.com/zPfEoVL.png">
 
  Với công nghệ này, trên một máy chủ vật lý, ta sẽ sinh ra được nhiều máy con (giống với công nghệ ảo hóa virtualization), nhưng tốt hơn ở chỗ là các máy con này (Guess OS) đều dùng chung phần nhân của máy mẹ (Host OS) và chia sẻ với nhau tài nguyên máy mẹ. Có thể nói là khi nào cần tài nguyên thì được cấp, cần bao nhiêu thì cấp bấy nhiêu, như vậy việc tận dụng tài nguyên đã tối ưu hơn.
## 3. Container là gì?
Container là một giải pháp để chuyển giao phần mềm một cách đáng tin cậy giữa các môi trường máy tính khác nhau bằng cách  tạo ra một môi trường chứa mọi thứ mà phần mềm cần để có thể chạy được mà không bị các yếu tố liên quan đến môi trường hệ thống làm ảnh hưởng tới cũng như không làm ảnh hưởng tới các phần còn lại của hệ thống.
*Ưu điểm*
- *Linh động*: Triển khai ở bất kỳ nơi đâu do sự phụ thuộc của ứng dụng vào tầng OS cũng như cơ sở hạ tầng được loại bỏ
- *Nhanh*: Do chia sẻ host OS nên container có thể được tạo gần như một cách tức thì
- *Nhẹ*: Container cũng sử dụng chung các images nên cũng không tốn nhiều disks.
- *Đồng nhất* :Khi nhiều người cùng phát triển trong cùng một dự án sẽ không bị sự sai khác về mặt môi trường.
*Nhược điểm*
- Container dùng chung OS kernel Linux nên nếu có vấn đề gì ảnh hưởng tới OS kernel của Node thì nó củng bị ảnh hưởng.
- Số lượng container càng lớn thì càng phức tạp.
## 4. Docker
 Docker là một ứng dụng mã nguồn mở cho phép đóng gói các ứng dụng, các phần mềm phụ thuộc lẫn nhau vào trong cùng một container. Container này sau đó có thể mang đi triển khai trên bất kỳ một hệ thống Linux phổ biến nào. Các container này hoàn toàn độc lập với các container khác.
 
 Những lợi ích mà Docker đem lại:
 
- Sử dụng ít tài nguyên: Thay vì phải ảo hóa toàn bộ hệ điều hành thì chỉ cần build và chạy các container độc lập sử dụng chung kernel duy nhất.
- Tính đóng gói và di động: Tất cả các gói dependencies cần thiết đều được đóng gói vừa đủ trong container. Và sau đó có thể mang đi triển khai trên các server khác.
- Cô lập tài nguyên: server bố không biết ở trong container chạy gì và container cũng không cần biết bố nó là CentOs hay Ubuntu =)). Các container độc lập với nhau và có thể giao tiếp với nhau bằng một interface
- Hỗ trợ phát triển và quản lý ứng dụng nhanh: Đối với Dev, sử dụng docker giúp họ giảm thiểu thời gian setup môi trường, đóng gói được các môi trường giống nhau từ Dev - Staging - Production :v
- Mã nguồn mở: Cộng đồng support lớn, các tính năng mới được release liên tục.