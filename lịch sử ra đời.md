# I. Lịch sử ra đời docker
## 1. Vitualization
Từ xưa, mô hình máy chủ thường là máy chủ vật lý + hệ điều hành (OS) + application.
<img src ="https://i.imgur.com/OCNuS4g.png">

Mô hình gặp phải vấn đề là vì một máy chủ chỉ cài được một OS nên gây ra việc không tận dụng hết tài nguyên, gây lãng phí. Đây là lý do công nghệ ảo hóa Vitualization ra đời.
<img src = "https://i.imgur.com/73esvQW.png">

Ngày nay áo hóa đã trở thành xu hướng chung của rất nhiều doanh nghiệp,
đặc biệt là các doanh nghiệp lớn trên thế giới. Virtualization giúp họ tiết
kiệm chi phí hiệu quả với khả năng tận dụng tối đa năng suất của các thiết
bị phần cứng. Ngoài ra vIệc áp dụng công nghệ ảo hóa máy chủ còn họ giúp
tiết kiệm không gian sử dụng, nguồn điện và giải pháp tỏa nhiệt trong trung
tâm dữ liệu, giảm thời gian thiết lập máy chủ, kiểm tra phần mềm trước khi
đưa vào hoạt động.
Ảo hóa là việc chia phần cứng vật lý thành nhiều phần cứng ảo. Vì vậy, có thể
nói ảo hóa là việc chia một máy vật lý thành nhiều máy con ảo.
Công nghệ ảo hóa thực hiện ảo hóa trên máy tính, bao gồm các kỹ thuật và quy trình thực hiện ảo hóa. Các kỹ thuật và quy trình này để tạo ra một tầng trung gian giữa hệ thống phần cứng máy tính và phần mềm chạy trên nó. Ý tưởng ban đầu của công nghệ ảo hóa là từ một máy vật lý đơn lẻ có thể tạo thành nhiều máy ảo độc lập. Nó cho phép tạo nhiều máy ảo trên một máy chủ vật lý, mỗi một máy ảo cũng được cấp phát tài nguyên phần cứng như máy thật gồm có RAM, CPU, Card mạng, ổ cứng, các tài nguyên khác và hệ điều hành riêng. Khi chạy ứng dụng, người sử dụng chỉ chú ý tới khái niệm logic về tài nguyên máy tính hơn là khái niệm vật lí về tài nguyên máy tính. 
Hệ thống máy chủ được thiết kế để chạy một hệ điều hành và một ứng dụng sẽ không khai thác triệt để hiệu năng của hầu hết các máy chủ rất lớn. Vì thế sự ra đời của ảo hóa cho phép ta vận hành nhiều máy chủ ảo trên cùng một máy chủ vật lý, dùng chung các tài nguyên của một máy chủ vật lý qua nhiều môi trường khác nhau. Các máy chủ ảo khác nhau có thể vận hành nhiều hệ điều hành và ứng dụng khác nhau trên cùng một máy chủ vật lý.
Tuy nhiên, công nghệ Vitualization cũng nảy sinh một vài vấn đề như:
	- Về tài nguyên: Khi bạn chạy máy ảo, bạn phải cung cấp "cứng" dung lượng ổ cứng cũng như ram cho máy ảo đó, bật máy ảo lên để đó không làm gì thì máy thật cũng phải phân phát tài nguyên. Ví dụ khi tạo một máy ảo ram 2GB trên máy thật ram 4GB, lúc này máy thật sẽ mất 2GB ram cho máy ảo, kể cả khi máy ảo không dùng hết 2GB ram, đó là một sự lãng phí
	- Về thời gian: Việc khởi động, shutdown khá lâu, có thể lên tới hàng phút.
Do đó, người ta sinh ra công nghệ Containerlization.

 