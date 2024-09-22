---
title: "Forget and Rewire: Enhancing the Resilience of 
Transformer-based Models against Bit-Flip Attacks"
layout: post
date: 22-09-2024
author: -Sukemcute
mermaid: true
math: true
categories: [NCKH, Paper]
tag: [Bit-flip, Resnet, DNN]
image: /assets/img/other/ForgetandRewire.png
description: Phân tích bài báo.
---

## Tóm tắt
- Tấn công Bit-Flip (BFA) liên quan đến việc đối thủ thao túng các bit tham số của mô hình để làm suy yếu đáng kể độ chính xác của nó. 
- Thường nhắm mục tiêu vào các tham số dễ bị tổn thương nhất, gây ra thiệt hại tối đa với các bit lật tối thiểu. Mặc dù tác động của BFA đối với Mạng nơ-ron sâu (DNN) được nghiên cứu kỹ lưỡng, nhưng tác động của chúng đối với Mô hình ngôn ngữ lớn (LLM) và Máy biến đổi tầm nhìn (ViT-Vision Transformer) không nhận được sự chú ý tương tự. Lấy cảm hứng từ "nối lại não" (brain rewiring), chúng tôi khám phá việc tăng cường khả năng phục hồi của Transformers trước các cuộc tấn công như vậy. Tiềm năng này nằm trong kiến trúc độc đáo của các mô hình dựa trên máy biến áp, đặc biệt là các lớp tuyến tính của chúng. 
- Cách tiếp cận mới lạ được gọi là **Forget and Rewire (FaR)**, áp dụng chiến lược nối lại cho các lớp tuyến tính để làm xáo trộn các kết nối tế bào thần kinh. Bằng cách phân phối lại các nhiệm vụ từ các tế bào thần kinh quan trọng đến không thiết yếu, giảm độ nhạy của mô hình đối với các thông số cụ thể trong khi vẫn giữ được chức năng cốt lõi của nó. 
  - Chiến lược này ngăn chặn nỗ lực của đối thủ trong việc xác định và nhắm mục tiêu các tham số quan trọng bằng cách sử dụng các thuật toán dựa trên gradient.
  - Che giấu các thông số quan trọng và tăng cường khả năng chống lại các cuộc tấn công ngẫu nhiên. Đánh giá toàn diện trên các bộ dữ liệu và khung Transformer được sử dụng rộng rãi cho thấy cơ chế FaR làm **giảm đáng kể** tỷ lệ thành công BFA từ **1,4 đến 4,2** lần với tổn thất độ chính xác tối thiểu (dưới 2%).

## **1. Giới thiệu**

LLM và ViTs đã chuyển đổi các nhiệm vụ trong xử lý ngôn ngữ tự nhiên và thị giác máy tính, thể hiện khả năng ấn tượng trong việc hiểu và tạo văn bản và phân loại hình ảnh. Tuy nhiên, các mô hình này dễ bị tổn thương trước các đối thủ do phần cứng gây ra, chẳng hạn như Tấn công Bit-Flip (BFA). BFA khai thác lớp phần cứng để đưa ra lỗi trong các vùng bộ nhớ lưu trữ các tham số trọng lượng của mô hình, ảnh hưởng đến tính toàn vẹn và hiệu suất của mô hình. Các kỹ thuật như DeepHammer nhắm mục tiêu các bit cụ thể trong trang DRAM, thay đổi trọng lượng nhạy cảm và làm giảm hiệu suất mô hình.

Mặc dù có những tiến bộ trong công nghệ bộ nhớ, các phương pháp mới vẫn có thể ảnh hưởng từ xa đến nội dung bộ nhớ mà không cần truy cập vật lý trực tiếp. Bảo vệ chống lại BFA rất phức tạp do thuật toán Tìm kiếm bit lũy tiến dựa trên độ dốc, xác định các bit nhạy cảm nhất trong trọng số của mô hình. Do đó, việc hiểu và giảm thiểu các cuộc tấn công này là rất quan trọng để đảm bảo tính mạnh mẽ và độ tin cậy của các mô hình tiên tiến này trong các ứng dụng trong thế giới thực. Mặc dù các công nghệ bộ nhớ hiện đại đã tăng cường khả năng phục hồi cho BFA, các nhà nghiên cứu tiếp tục khám phá các phương pháp mới có khả năng ảnh hưởng đến nội dung của bộ nhớ từ xa, mà không cần truy cập vật lý trực tiếp. Gắn kết một phòng thủ chống lại BFA vốn là một nhiệm vụ phức tạp. Sự phức tạp này phát sinh từ việc BFA sử dụng thuật toán Tìm kiếm bit lũy tiến dựa trên độ dốc, xác định tỉ mỉ và sửa đổi các bit trong các trọng lượng nhạy cảm nhất của mô hình.

Lấy cảm hứng từ nhà tâm lý học thần kinh nổi tiếng Donald Hebb, người đã đặt ra cụm từ nổi tiếng “Các tế bào thần kinh cùng hoạt động, kết nối với nhau”, chúng tôi khám phá ra sự tương đồng trong khả năng thích ứng và khả năng phục hồi của bộ não con người khi đối mặt với những thách thức. Tuyên bố phổ biến này nhấn mạnh khả năng của bộ não trong việc tổ chức lại và nối lại, thích nghi với các hoạt động, suy nghĩ và cảm xúc của chúng ta trong một quá trình năng động. Tương tự, nếu một số kiểu suy nghĩ nhất định không còn phục vụ lợi ích tốt nhất của chúng ta, bộ não của chúng ta có khả năng điều chỉnh lại để áp dụng những suy nghĩ thay thế, thể hiện khả năng thay đổi đáng chú ý.

Trong nghiên cứu này, chúng tôi lấy cảm hứng từ các cơ chế thích ứng của não để giải quyết các lỗ hổng tương tự trong các kiến trúc dựa trên máy biến áp. Đề xuất của chúng tôi tập trung vào việc phân bổ lại các tham số dư thừa trong các mô hình dựa trên máy biến áp như LLM và ViT, chuyển hướng chúng để tăng cường các thông số quan trọng hơn. Chiến lược này đảm bảo hoạt động tập thể của các tham số ít được sử dụng hơn với các tham số then chốt, tăng cường khả năng phục hồi của mô hình chống lại các đối thủ do phần cứng gây ra như BFA, trong khi vẫn duy trì độ chính xác với tổn thất không đáng kể.

Lấy tín hiệu từ các chiến thuật được sử dụng bởi những kẻ tấn công, công việc này giới thiệu một cơ chế phòng thủ mới nhằm làm xáo trộn các trọng lượng nhạy cảm nhất của các mô hình dựa trên máy biến áp. Phương pháp được đề xuất xác định các tham số nhạy cảm nhất và ít nhạy cảm nhất trong mô hình, sau đó nối lại các tham số ít nhạy cảm nhất với các tham số được coi là quan trọng nhất. Việc phân phối lại trách nhiệm tác vụ trên các tham số này làm giảm hiệu quả độ dốc độ nhạy, khiến các thuật toán tìm kiếm dựa trên gradient kém hiệu quả hơn trong việc xác định các tham số chính. Về cơ bản, cách tiếp cận này nhằm mục đích che giấu các tham số quan trọng này, đảm bảo rằng bất kỳ tiêm bit-flip nào cũng nhắm vào các tham số ít quan trọng hơn.

Chúng tôi trình bày một khám phá kỹ lưỡng về các khía cạnh lý thuyết và thực tiễn của BFA, cụ thể, xem xét các ràng buộc trong thế giới thực của LLM và ViTs. Dựa trên những hiểu biết này, chúng tôi giới thiệu một khuôn khổ (Forget and Rewire: FaR) dành riêng cho việc xác định chính xác các thông số nhạy cảm và tăng cường khả năng phục hồi của chúng trước các cuộc tấn công thích ứng.

Các đánh giá toàn diện của chúng tôi cho thấy rằng cách tiếp cận được đề xuất của chúng tôi, trong khi ảnh hưởng tối thiểu đến độ chính xác của mô hình (-1,97% trên ImageNet mà không cần đào tạo lại), làm xáo trộn đáng kể trọng số quan trọng lên đến 84% và 91% từ các chuyên gia tiềm năng và những kẻ tấn công cơ bản, tương ứng. Đồng thời, FaR buộc Oracle và những kẻ tấn công cơ bản thực hiện nhiều bit hơn 1,6 lần lên đến 4 lần để đạt được mức độ gián đoạn tương tự (suy giảm độ chính xác hơn 10%) như khi không có tăng cường FaR. Để đạt được độ bền chấp nhận được so với BFA bằng cách làm xáo trộn 15% tham số trên mỗi lớp, FaR phát sinh chi phí tương ứng lên tới 4,5% và 9,3% trong thời gian suy luận và lưu trữ mô hình.

Điều đáng chú ý là cơ chế phòng thủ được đề xuất của chúng tôi thể hiện khả năng tương thích đáng kể, cho phép nó phối hợp với các chiến lược phòng thủ khác, chẳng hạn như NeuroPots và Aegis. Khả năng tương thích này được cho là do FaR tập trung chính vào việc tăng cường sức mạnh tổng thể của các mô hình thay vì chỉ dựa vào phát hiện và phục hồi tấn công.

Theo hiểu biết tốt nhất của chúng tôi, đây là công trình đầu tiên áp dụng phương pháp nối lại để che giấu các tham số quan trọng của các mô hình dựa trên máy biến áp, cải thiện khả năng phục hồi của chúng trước các cuộc tấn công bit-flip tập trung vào mô hình đầy thách thức. Những cuộc tấn công này đặt ra một thách thức đáng kể với bề mặt tấn công có khả năng quá rộng để các giải pháp hiện có có thể bao quát toàn diện. Cuối cùng nhưng không kém phần quan trọng, thư viện FaR có nguồn mở và có sẵn công khai cho cộng đồng để phát triển và đánh giá. Chúng tôi hy vọng rằng kết quả của chúng tôi có thể cung cấp một viễn cảnh mới để bảo vệ chống lại các cuộc tấn công mới nổi trong học sâu, thúc đẩy nghiên cứu chuyên sâu hơn theo hướng này.

## **2. Background** 

Trong phần này, chúng tôi trình bày một cái nhìn tổng quan ngắn gọn về kiến thức sơ bộ cần thiết, đặc biệt là về Transformers, biểu diễn dấu phẩy động trong phần cứng hiện đại và các cuộc tấn công bit-flip.

### **2.1. Transformer-based Models**

Kiến trúc Transformer tạo thành xương sống của Mô hình ngôn ngữ lớn và Máy biến áp tầm nhìn, đóng một vai trò quan trọng trong các đường ống hiện đại, tiên tiến để xử lý ngôn ngữ tự nhiên và phân loại hình ảnh. Trong phần này, chúng tôi cung cấp chi tiết về kiến trúc Transformer , nhấn mạnh tính tương thích và hiệu quả của phương pháp tiếp cận của chúng tôi khi áp dụng cho lớp mô hình này. 
Mô hình Transformer có hai phần chính: bộ mã hóa và bộ giải mã, như thể hiện trong **Hình 1.** Bộ mã hóa lấy một đầu vào và biến nó thành một đại diện (tính năng). Điều này giúp mô hình hiểu được đầu vào. Mặt khác, bộ giải mã sử dụng biểu diễn từ bộ mã hóa và một số thông tin khác để tạo chuỗi đầu ra. Một trong hai khối này có thể được sử dụng độc lập dựa trên bản chất của nhiệm vụ trong tầm tay:
  - **Chỉ bộ mã hóa (Encoder-only):** Thích hợp cho các tác vụ yêu cầu hiểu biết về đầu vào, chẳng hạn như phân loại văn bản và nhận dạng thực thể được đặt tên (NER). 
  - **Chỉ giải mã (Decoder-only):** Thích hợp cho các tác vụ phát sinh.  
  - **Bộ mã hóa-giải mã (Encoder-decoder):** Thích hợp cho các tác vụ phát sinh yêu cầu đầu vào, chẳng hạn như tóm tắt hoặc dịch.

Một đặc điểm chính của các mô hình Biến áp nằm ở sự kết hợp của các lớp chuyên biệt được gọi là các lớp chú ý. Các lớp này hướng dẫn mô hình nhấn mạnh đặc biệt đến các từ cụ thể trong một câu, ưu tiên chúng một cách hiệu quả trong khi giảm nhấn mạnh những từ khác khi xây dựng biểu diễn của mỗi từ. Mặt nạ chú ý cũng có thể được sử dụng trong bộ mã hóa / bộ giải mã để ngăn người mẫu chú ý đến một số từ đặc biệt. 

Đi sâu vào lớp chú ý nhiều đầu, Nó bao gồm một số mô-đun Tuyến tính, mỗi mô-đun có bộ trọng lượng riêng. Trong lớp chú ý, có ba tham số thiết yếu: Query, Key và Value. Ba lớp tuyến tính riêng biệt này, có chung cấu trúc tương tự, coi mỗi từ trong chuỗi là một vector. Biểu diễn đầu vào được mã hóa được truyền qua cả ba tham số. Quá trình này dẫn đến một biểu diễn được mã hóa cập nhật bao gồm điểm chú ý cho mỗi từ. 

Trong kiến trúc Transformer , mô-đun Attention thực hiện các tính toán song song nhiều lần. Mỗi tính toán song song này được gọi là Đầu chú ý. Mô-đun Attention chia các tham số Query, Key và Value thành N phần riêng biệt, xử lý từng phần tách này một cách độc lập thông qua Đầu chuyên dụng của riêng nó. Sau khi các tính toán Chú ý riêng lẻ này, có chung cấu trúc tương tự, được hoàn thành, kết quả của chúng được kết hợp để tạo ra điểm Chú ý cuối cùng. Cách tiếp cận này được gọi là sự chú ý Multihead, và nó trao quyền cho Transformer để nắm bắt một phạm vi rộng hơn của các mối quan hệ và sự tinh tế liên quan đến mỗi từ, nâng cao năng lực tổng thể của nó.
![Hình 1: Architecture of Transformers](/assets/img/other/forgetnrewire/Hinh1.png)
_Hình 1: Kiến trúc của Transformer_

### **2.2. IEEE 754 Floating Point Arithmetic (Số học dấu phẩy động)**

Các thông số trọng lượng thường được biểu diễn bằng cách sử dụng số dấu phẩy động có độ chính xác đơn 32 bit của IEEE-754. Định dạng này khai thác ký hiệu hàm mũ nhưng hy sinh độ chính xác cho một phạm vi rộng hơn các giá trị có thể. Ví dụ: giá trị 0,34375 trong ký hiệu hàm mũ được biểu thị bằng $ 1,375×2^{-2} $, trong đó 1,375 là bọ ngựa và -2 là số mũ. Định dạng dấu phẩy động có độ chính xác đơn IEEE 754 phân bổ 23 bit cho bọ ngựa, 8 bit cho số mũ và một bit cho dấu của giá trị. Điều làm cho định dạng này hấp dẫn từ quan điểm đối nghịch là tác động khác nhau của các bit khác nhau đối với giá trị được biểu diễn. Ví dụ, hãy xem xét lật bit thứ 20 trong bọ ngựa, dẫn đến tăng nhẹ từ 0,34375 lên 0,359375, tạo thành một nhiễu loạn không quan trọng điển hình. Ngược lại, lật bit mũ cao nhất biến đổi giá trị thành $ 1.375 × 2^{126} $. Mặc dù cả hai kịch bản đều liên quan đến thao tác một bit, nhưng chúng tạo ra hai kết quả khác nhau.

### **2.3. Bit-Flip Attacks**

Sự mạnh mẽ, bảo mật và an toàn của một hệ thống máy tính hiện đại phụ thuộc vào sự cô lập bộ nhớ được thực thi ở cấp độ phần mềm và phần cứng. Tuy nhiên, ngay cả trong các chip DRAM hiện đại, sự cô lập bộ nhớ có thể bị xâm phạm do nhiễu độ đọc. Một ví dụ đáng chú ý về điều này là **RowHammer**, một hiện tượng nhiễu đọc được nghiên cứu rộng rãi. Trong RowHammer, hành động truy cập lặp đi lặp lại và đóng (búa) một hàng DRAM nhiều lần dẫn đến bitflips xảy ra trong các hàng gần nhau. Một ví dụ khác là **RowPress**, cuộc tấn công tiêm lỗi bộ nhớ gần đây nhất nhắm vào các hệ thống dựa trên DDR4 được củng cố bởi phần cứng bảo vệ RowHammer. RowPress đã chỉ ra rằng ngay cả một chương trình cấp người dùng vẫn có thể vi phạm sự cô lập bộ nhớ bằng cách duy trì trạng thái mở trong hàng DRAM trong một thời gian dài, dẫn đến sự gián đoạn trong các hàng liền kề vật lý đủ đáng kể để gây ra bitflips.

![Hình 2](/assets/img/other/forgetnrewire/Hinh2.png)
_Hình 2: Tổng quan về tấn công Bit-Flip_

Mục tiêu chính của các cuộc tấn công bit-flip là sử dụng bất kỳ cuộc tấn công tiêm lỗi bộ nhớ thực tế nào để giảm độ chính xác của mô hình trong khi giảm thiểu số lượng bit-flip được thực hiện. Mục đích của kẻ tấn công tàng hình là phân loại lại các đầu vào ban đầu thuộc loại nguồn p thành một loại mục tiêu q khác (trong đó q không bằng p) trong khi đảm bảo rằng các đầu vào còn lại duy trì các loại ban đầu của chúng, do đó duy trì độ chính xác của chúng và đảm bảo cuộc tấn công vẫn tàng hình.

Một trong những BFA thành công nhất là cuộc tấn công DeepHammer giúp tăng cường BFA bằng cách tinh chỉnh thuật toán được sử dụng để tìm kiếm các bit dễ bị tổn thương. DeepHammer cũng sử dụng **Progressive Bit Search (PBS)** dựa trên gradient để tìm các bit dễ bị tấn công, như thể hiện trong **Hình 2.** Trong cuộc tấn công này, trong lần lặp thứ k, DeepHammer trước tiên chọn n bit dễ bị tổn thương dựa trên xếp hạng gradient của các tham số của mô hình. Sau đó, DeepHammer lật riêng từng bit này, tạo ra một tập hợp tổn thất được ký hiệu là *L*. Quá trình này được lặp lại cho mỗi lớp, dẫn đến tổng số bit ứng cử viên *n×l* và bộ tổn thất tương ứng của chúng. DeepHammer xác định bit dễ bị tổn thương nhất là bit có tổn thất cao nhất. Bit-flip được thực hiện lặp đi lặp lại cho đến khi đạt được kết quả mong muốn. Mặc dù DeepHammer bị giới hạn chỉ lật một bit trên mỗi trang, chúng tôi giả định rằng đối thủ có khả năng lật bất kỳ số bit nào trên mỗi trang DRAM.

## **3. Motivation (Động lực)

### **3.1. Thách thức và mối đe dọa bảo mật trên LLM và ViTs

Sự phổ biến của các mô hình dựa trên máy biến áp như LLM và ViTs đã tăng lên đáng kể, đánh dấu một sự thay đổi đáng kể trong bối cảnh học máy. Những mô hình tinh vi này, được minh họa bằng các kiến trúc máy biến áp, đã tìm thấy ứng dụng rộng rãi trên nhiều lĩnh vực, từ dịch máy và tạo nội dung đến trợ lý ảo và phân tích dữ liệu. Tuy nhiên, sự phổ biến này đã đưa lên hàng đầu một loạt các mối quan tâm về an ninh. Sức mạnh to lớn của LLM để tạo ra văn bản mạch lạc và phù hợp với ngữ cảnh có thể được khai thác bởi các tác nhân độc hại cho các mục đích như phổ biến thông tin sai lệch, lừa đảo tự động và thậm chí tạo ra nội dung giả mạo thuyết phục. Ngoài ra, LLM và ViTs không tránh khỏi các cuộc tấn công đối nghịch, trong đó các sửa đổi đầu vào tinh tế có thể thao túng đầu ra của chúng hoặc làm tổn hại đến quá trình ra quyết định của chúng. Khi LLM và ViTs được tích hợp sâu hơn vào các hệ thống quan trọng, tác động tiềm tàng của các lỗ hổng bảo mật này trở nên rõ rệt hơn. Giải quyết những mối quan tâm này là rất quan trọng để đảm bảo rằng lợi ích của các mô hình dựa trên máy biến áp được khai thác mà không làm suy yếu tính toàn vẹn dữ liệu và quyền riêng tư trong bối cảnh kỹ thuật số.

Các cuộc tấn công bảo mật vào các mô hình học máy có thể được phân loại như sau: 
  **1)** Các cuộc tấn công trốn tránh, liên quan đến việc nhiễu loạn đầu vào trong quá trình thử nghiệm để đánh lừa mô hình phân loại; 
  **2)** Các cuộc tấn công đầu độc, nhằm thao túng các bộ dữ liệu đào tạo để mang lại các mô hình ML được đào tạo kém; 
  **3)** Các cuộc tấn công tiêm lỗi, sửa đổi các tham số ML để thay đổi phân loại các đầu vào cụ thể đối với nhãn đích. Bất kể loại tấn công nào, mục tiêu bao trùm của các cuộc tấn công đối thủ là gây ra sự phân loại sai trong một số đầu vào nhất định, trong khi vẫn duy trì độ chính xác mô hình cao để những người khác vẫn tàng hình.

### **3.2. Giải quyết khoảng cách nghiên cứu**

Trong khi nhiều nghiên cứu đã khám phá các lỗ hổng của DNN đối với các cuộc tấn công lỗi hoặc tính nhạy cảm của LLM và ViTs đối với các cuộc tấn công trốn tránh, một lỗ hổng đáng kể trong nghiên cứu vẫn chưa được giải quyết: điều tra các cuộc tấn công phun lỗi nhắm mục tiêu cụ thể vào các mô hình dựa trên máy biến áp. Các cuộc tấn công lỗi vào DNN đã tập trung vào việc khai thác các lỗ hổng trong việc triển khai phần cứng của chúng hoặc thao túng các tham số của chúng để gây ra phân loại sai. Tương tự, các cuộc tấn công trốn tránh vào LLM và ViTs đã đào sâu vào việc tạo ra các đầu vào đối nghịch để đánh lừa quá trình ra quyết định của các mô hình. Tuy nhiên, lĩnh vực khác biệt của các cuộc tấn công tiêm lỗi vào LLM và ViTs, liên quan đến việc giới thiệu các lỗi bộ nhớ được kiểm soát hoặc nhiễu loạn tham số để thao túng đầu ra của chúng, vẫn chưa được khám phá. Lãnh thổ chưa được khám phá này đưa ra một thách thức độc đáo, do tính chất phức tạp của kiến trúc dựa trên máy biến áp và các ứng dụng đa dạng của nó. Các cuộc tấn công tiêm lỗi vào LLM và ViTs có khả năng dẫn đến việc tạo văn bản ngoài ý muốn hoặc sai lệch, hoặc phân loại sai, làm suy yếu sự gắn kết của chúng hoặc thậm chí cho phép các thao tác tinh vi khó phát hiện.

Với mục đích khám phá tính nhạy cảm của LLM và ViTs đối với các cuộc tấn công lỗi, chúng tôi đã bắt tay vào một cuộc điều tra. Mục tiêu chính của chúng tôi là đánh giá khả năng phục hồi của các mô hình này bằng cách phân tích các thành phần và tham số mô hình riêng lẻ thông qua việc giới thiệu bit-flip. Chúng tôi đã tìm cách hiểu làm thế nào những lỗi gây ra này có khả năng ảnh hưởng đến đầu ra phân loại, vẫn không thể nhận ra đối với các nhà quan sát của con người. Để theo đuổi mục tiêu này, chúng tôi đã đưa các thông số máy biến áp vào các BFA mới nhất.

Phân tích của chúng tôi đã tiết lộ một quan sát quan trọng: trong LLM và ViTs, một số thông số nhất định xuất hiện như gót chân Achilles. Các tham số chọn lọc này đại diện cho các điểm lỗi đơn lẻ trong mô hình, trong đó ngay cả một bit-flip (SBF) cũng có thể gây ra sự suy giảm hiệu suất nghiêm trọng. Quan sát này được hỗ trợ bởi những phát hiện từ Tài liệu tham khảo, trong đó lưu ý rằng các bit lật đơn trong một số tham số không phải Wquery và Wkey nhất định có thể làm gián đoạn nghiêm trọng đầu ra bản dịch, làm giảm điểm BLEU xuống gần bằng không. Các cuộc tấn công lật một bit, nếu thành công, sẽ dẫn đến lỗi mô hình hoàn toàn (ví dụ: độ chính xác giảm xuống 0), thường được biểu thị bằng các giá trị "NaN" trong tenxơ PyTorch. Vì hành vi này có thể dễ dàng phát hiện, nó không phải là một cách tiếp cận yêu thích của những kẻ tấn công. Điều quan trọng cần nhấn mạnh là BFA là đối thủ tàng hình hơn có khả năng lật nhiều bit trên mỗi trang DRAM để làm giảm hiệu suất mà không bị phát hiện. Hình 3 cho thấy một ví dụ về phân tích của chúng tôi về SBF trên mô hình roberta-large-mnli từ thư viện ôm mặt. Chúng tôi đã chọn ngẫu nhiên 100 tham số trên mỗi lớp của bộ mã hóa máy biến áp và điều tra xem SBF có thể thay đổi đầu ra phân loại văn bản hay không. Quan sát minh họa rằng dense3 và dense2 là các mô-đun dễ bị tổn thương nhất trong khi các mô-đun chuẩn hóa không cho thấy bất kỳ lỗ hổng nào. 

Ngược lại, chúng tôi cũng quan sát thấy sự tồn tại của nhiều tế bào thần kinh và các thông số trong mô hình vẫn không hoạt động, tạo ra các kích hoạt không đáng kể mà không ảnh hưởng đến độ nhạy của mô hình. Quan sát này khiến chúng tôi suy ngẫm về một cách tiếp cận sáng tạo nhằm tận dụng các thông số không cần thiết này để tăng cường độ bền của mô hình so với BFA.

![Hình 3](/assets/img/other/forgetnrewire/Hinh3.png)
_Hình 3: Phân tích lỗ hổng của các lớp và mô-đun biến áp thành BFA đơn_

















## Giải thích từ chuyên môn
### 1. ViT (Vision Transformer): Máy biến đổi tầm nhìn
[https://fineproxy.org/vi/wiki/vit-vision-transformer/](https://fineproxy.org/vi/wiki/vit-vision-transformer/)
### 2. LLMs (Large language model): Mô hình ngôn ngữ lớn
[https://aws.amazon.com/vi/what-is/large-language-model/](https://aws.amazon.com/vi/what-is/large-language-model/)
### 3. DeepHammer
[https://www.usenix.org/system/files/sec20-yao.pdf](https://www.usenix.org/system/files/sec20-yao.pdf)
### 4. DRAM (Dynamic Random Access Memory): Bộ nhớ truy cập ngẫu nhiên động
[https://dienthoaivui.com.vn/tin-tuc/dram-la-gi-sram-la-gi](https://dienthoaivui.com.vn/tin-tuc/dram-la-gi-sram-la-gi)
