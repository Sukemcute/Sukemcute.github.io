---
title: "Defending and Harnessing the Bit-Flip based Adversarial Weight Attack"
layout: post
date: 25-09-2024
author: -Sukemcute
mermaid: true
math: true
categories: [NCKH, Paper]
tag: [Bit-flip, Resnet, DNN]
image: /assets/img/other/defending/Hinh.png
description: Phân tích bài báo.
---

## Tóm tắt
Gần đây, một mô hình mới của cuộc tấn công đối thủ vào trọng lượng mạng thần kinh lượng tử hóa đã thu hút sự chú ý lớn, đó là cuộc tấn công trọng lượng đối thủ dựa trên Bit-Flip, hay còn gọi là. Tấn công Bit-Flip (BFA). BFA đã cho thấy khả năng tấn công phi thường, trong đó đối thủ có thể làm trục trặc Mạng nơ-ron sâu (DNN) lượng tử hóa như một dự đoán ngẫu nhiên, thông qua các bit bit độc hại trên một tập hợp nhỏ các bit trọng lượng dễ bị tổn thương (ví dụ: 13 trong số 93 triệu bit của ResNet-18 lượng tử hóa 8 bit). Tuy nhiên, không có phương pháp phòng thủ hiệu quả nào để tăng cường khả năng chịu lỗi của DNN chống lại BFA đó. Trong công việc này, chúng tôi tiến hành điều tra toàn diện về BFA và đề xuất tận dụng đào tạo nhận thức nhị phân và thư giãn của nó - phân cụm theo từng phần như các biện pháp đối phó đơn giản và hiệu quả đối với BFA. Các thí nghiệm cho thấy, để BFA đạt được sự suy giảm độ chính xác dự đoán giống hệt nhau (ví dụ: dưới 11% trên CIFAR-10), nó yêu cầu các bit lật độc hại hiệu quả hơn 19,3× và 480,1× trên ResNet-20 và VGG-11 tương ứng, so với các đối tác không phòng thủ.

## **1. Giới thiệu**

Khi Mạng nơ-ron sâu (DNN) đạt được hiệu suất vượt trội của con người trong nhiều tác vụ liên quan đến thị giác máy tính, các ứng dụng của nó trong các tình huống trong thế giới thực đang phát triển nhanh chóng. Trong kịch bản như vậy, khả năng chịu lỗi của mạng thần kinh là mối quan tâm nghiên cứu lớn để phát triển các mạng thần kinh đáng tin cậy chống lại lỗi ngẫu nhiên yếu và thậm chí các cuộc tấn công độc hại mạnh. Một số lượng đáng kể nỗ lực nghiên cứu đã tập trung vào DNN bị đánh lừa bởi tiếng ồn đầu vào không thể nhận ra của con người, hay còn gọi là. ví dụ đối nghịch. Tuy nhiên, một khía cạnh dễ bị tổn thương khác của DNN là các tham số mô hình, hầu như không được nghiên cứu. 

Do kích thước mô hình khổng lồ (hàng trăm MB cho DNN hiện đại), các bộ tăng tốc DNN hiện đại (ví dụ: GPU) thường cần lưu trữ các tham số mô hình trong bộ nhớ chính, cụ thể là DRAM Bộ nhớ truy cập ngẫu nhiên động. Những tiến bộ nghiên cứu gần đây đã đưa ra vấn đề lỗ hổng của dữ liệu được lưu trữ trong DRAM, nơi RowHammer Attack (RHA) đã được chứng minh là lật các bit bộ nhớ trong DRAM một cách độc hại mà không được cấp bất kỳ đặc quyền ghi dữ liệu nào, như được mô tả trong Hình 1. Thật không may, các DNN được lưu trữ trong DRAM với biểu diễn dấu phẩy động có thể dễ dàng bị tấn công để trục trặc hoàn toàn, thông qua lật bit đơn (ví dụ: trong một bit mũ có trọng lượng bất kỳ) thông qua RHA. Nhờ kỹ thuật lượng tử hóa trọng lượng DNN, DNN nhỏ gọn hơn vì các trọng số được thể hiện ở định dạng điểm cố định với biểu diễn bị hạn chế. Một đại diện như vậy đã được chứng minh là tăng cường đáng kể khả năng miễn dịch của DNN lượng tử hóa đối với các bit lật độc hại như vậy. Tuy nhiên, một cuộc tấn công Bit-Flip mới được đề xuất (BFA) có thuật toán tìm kiếm bit lũy tiến có thể xác định thành công và lật một số lượng cực nhỏ các bit trọng lượng dễ bị tổn thương (ví dụ: 13 trong số 93 triệu bit ResNet-18 trên ImageNet) để làm giảm độ chính xác suy luận DNN lượng tử hóa 8 bit quy mô lớn xuống mức thấp như dự đoán ngẫu nhiên (tức là từ 69,8% xuống 0,1%). Cho đến nay, vẫn còn thiếu các phương pháp phòng thủ hiệu quả chống lại BFA như vậy, và vì vậy chúng tôi đề xuất một biện pháp đối phó BFA dựa trên việc sử dụng nhị phân trọng lượng và thư giãn của nó - phân cụm theo mảnh. Những đóng góp trong thi: 

  • Một cuộc điều tra toàn diện về tấn công trọng lượng đối nghịch dựa trên bit-flip (tức là BFA) được tiến hành và thu được một số quan sát sâu sắc để hiểu lỗ hổng tham số đối với các cuộc tấn công này. 

  • Nhị phân trọng lượng và phương pháp thư giãn phân cụm theo từng phần của nó được đề xuất là các kỹ thuật phòng thủ hiệu quả chống lại BFA. 

  • Các phương pháp phòng thủ tấn công đối thủ bổ sung (ví dụ: huấn luyện đối thủ, cắt tỉa) và các phương pháp chính quy hóa mô hình thông thường cũng được kiểm tra.

## **2. Bối cảnh và các công trình liên quan** 

### **2.1. Tấn công trọng lượng đối thủ dựa trên Bit-Flip**

Cuộc tấn công trọng lượng đối thủ dựa trên bit-flip, hay còn gọi là. BitFlip Attack (BFA) [17], là một biến thể tấn công đối thủ thực hiện tiêm lỗi trọng lượng thông qua việc lật các bit. Đối với mục đích không thể nhận ra của máy, BFA chỉ lật các bit trọng lượng dễ bị tổn thương nhất được xác định bởi thuật toán Tìm kiếm bit lũy tiến (PBS) với tìm kiếm liên lớp và nội bộ lặp đi lặp lại. Cho một DNN lượng tử hóa nq-bit được tham số hóa bởi các bit (tức là trọng số lượng tử hóa trong hệ nhị phân), định nghĩa tenxơ {Bl} L l = 1, trong đó l ∈ {1, 2, ..., L} là chỉ số lớp. Tìm kiếm lớp nội bộ xác định bit có gradient cao nhất (arg maxBl |∇BlL|) là ứng cử viên bit dễ bị tổn thương, trong đó L là mất suy luận. Sau đó, tìm kiếm giữa các lớp so sánh các ứng cử viên bit được chọn bởi tìm kiếm bên trong lớp thông qua việc kiểm tra trực tiếp mức tăng tổn thất. Do đó, tìm kiếm bit trong lần lặp i có thể được xây dựng như một quá trình tối ưu hóa [17]:


### **2.2. Phòng thủ chống lại ví dụ đối nghịch**

s BFA là một biến thể tấn công đối thủ, các kỹ thuật phổ biến được sử dụng để bảo vệ ví dụ đối thủ [5] được nghiên cứu để tìm kiếm phương pháp phòng thủ BFA tiềm năng. Huấn luyện đối thủ. Huấn luyện đối kháng [5, 15] cho đến nay là phương pháp phòng thủ ví dụ đối thủ thành công nhất, tối ưu hóa các tham số DNN θ w.r.t cả đầu vào sạch x và ví dụ đối thủ của chúng xˆ như: min θ L (f (x; θ), t) + α · L(f(xˆ; θ), t ̃) (2) trong đó α là siêu tham số để cân bằng độ chính xác của mô hình được đào tạo trên dữ liệu tự nhiên sạch và các ví dụ đối nghịch. t ̃ là nhãn mềm như trong Eq. (1). Huấn luyện đối nghịch như vậy cũng thường được coi là một kỹ thuật chính quy hóa mạnh mẽ. Tăng công suất mô hình. Các công trình trước đây [15, 7] đã xác nhận bằng thực nghiệm sự cải thiện khả năng chống lại cuộc tấn công của đối thủ bằng cách tăng công suất mô hình. Nó được hiểu là các phân loại mạnh mẽ sẽ yêu cầu một ranh giới quyết định phức tạp hơn [15], điều này được kỳ vọng sẽ có lợi cho việc bảo vệ chống lại sự thay đổi trọng lượng độc hại. Phân tích chuyên sâu hơn về công suất mô hình và điện trở BFA được thảo luận trong Phần 6.

## **3. DNN theo BFA 101**

Để hiểu trước tiên, sau đó để bảo vệ và khai thác cuộc tấn công trọng lượng đối thủ dựa trên bitflip, chúng tôi đã tiến hành một số điều tra sơ bộ, cùng với một số quan sát quan trọng như được mô tả dưới đây.


Hình 2: Sự thay đổi trọng lượng do BFA gây ra cho (a) ResNet-20 [6] và (b) VGG-11 [20] trên tập dữ liệu CIFAR-10. Đối với cả hai kiến trúc, 5 thử nghiệm được thực hiện với các hạt giống ngẫu nhiên khác nhau. Mỗi chấm màu mô tả sự thay đổi trọng lượng (trục x: trọng lượng tấn công trước, trục y: trọng lượng sau tấn công) w.r.t một lần lặp lại của BFA. Thanh màu cho biết độ chính xác tương ứng (%) trên dữ liệu thử nghiệm CIFAR-10. Khoảng cách thẳng đứng giữa dấu chấm và đường đứt nét chéo (tức là y = x) thể hiện cường độ dịch chuyển trọng lượng. Hơn nữa, kết quả được báo cáo trong hình này sử dụng lượng tử hóa cân nặng sau đào tạo 8 bit [17].