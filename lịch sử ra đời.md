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

 <img src= "https://i.imgur.com/zPfEoVL.png")
 
  Với công nghệ này, trên một máy chủ vật lý, ta sẽ sinh ra được nhiều máy con (giống với công nghệ ảo hóa virtualization), nhưng tốt hơn ở chỗ là các máy con này (Guess OS) đều dùng chung phần nhân của máy mẹ (Host OS) và chia sẻ với nhau tài nguyên máy mẹ. Có thể nói là khi nào cần tài nguyên thì được cấp, cần bao nhiêu thì cấp bấy nhiêu, như vậy việc tận dụng tài nguyên đã tối ưu hơn.
 