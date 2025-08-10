## Giới thiệu

Trong dự án lần này, em và đồng đội trong vai trò Data Scientist sẽ thực hiện xây dựng một quy trình hoàn chỉnh (end-to-end) để dự đoán giá nhà dựa trên dữ liệu thu thập từ trang web bất động sản. Dự án được chia thành nhiều bước, bao gồm cào dữ liệu (scraping), trích xuất đặc trưng từ mô hình LLM, làm sạch, và phân tích dữ liệu cũng như xây dựng mô hình dự đoán.

Nhiệm vụ của em trong dự án này với vai trò Data Analyst:
- Chuẩn hóa giá trị trong các cột về một dạng nhất quán.
- Làm sạch dữ liệu từ hai DataFrame và áp dụng các hàm để xử lý các giá trị thiếu hoặc không hợp lệ.
- Khám phá dữ liệu giá nhà các quận/huyện trên khu vực thành phố Hồ Chí Minh nhằm rút ra những thông tin giá trị.
- Sử dụng các phương pháp kiểm định, lựa chọn các đặc trưng quan trọng cho việc xây dựng mô hình.
- Hỗ trợ đồng đội xây dựng và đánh giá mô hình.

## Cấu trúc thư mục và mô tả các tệp tin chính

- **data_crawled_batdongsan/** – Thư mục chứa toàn bộ file .txt của từng trang đã được cào từ trang bất động sản.
- **codeCrawl.ipynb** – Notebook chứa logic cào dữ liệu từ trang web (sử dụng thư viện requests, BeautifulSoup, v.v.).
- **raw_data.csv** – Dữ liệu thô sau khi trích xuất (chưa làm sạch).
- **implement_llm.ipynb** – Notebook triển khai mô hình LLM (Large Language Model) thông qua API (ví dụ: AnyScale) để trích xuất đặc trưng từ dữ liệu văn bản.
- **raw_data_llm.csv** – Lưu các đặc trưng do mô hình LLM trả về.
- **final_raw_data.csv** – Dữ liệu cuối cùng sau khi hợp nhất raw_data.csv và raw_data_llm.csv (chưa qua các bước xử lý tiếp theo).
- **HOUSING_PRICE_PREDICTION.ipynb** – Notebook chính, vừa chứa code vừa đóng vai trò báo cáo: làm sạch dữ liệu, xây dựng mô hình, đánh giá kết quả, v.v.
- **coordinates.csv** – Danh sách tọa độ (kinh độ, vĩ độ) các quận/huyện phục vụ cho việc mapping hoặc phân tích không gian.
- **quan_huyen.geojson** – File GeoJSON chứa tọa độ (đa giác) ranh giới quận/huyện, dùng để trực quan hóa trên bản đồ.
- **EDA_TEST_ON_RAW_DATA.html** – Báo cáo EDA (phân tích dữ liệu thô) trên dữ liệu ban đầu.
- **EDA_TEST_ON_CLEANED_DATA.html** – Báo cáo EDA trên dữ liệu đã làm sạch, dùng để so sánh và đánh giá chất lượng xử lý.

## Quy trình

### 1. Thu thập dữ liệu

- Thu thập dữ liệu từ trang web batdongsan.vn và tổ chức thông tin chi tiết về bất động sản vào một tệp CSV để dễ dàng quản lý và phân tích.Tự động hóa quá trình thu thập và xử lý dữ liệu từ nhiều trang web bất động sản. Tạo ra một cơ sở dữ liệu dễ dàng truy cập và sử dụng để phân tích thị trường bất động sản. Tối ưu hóa thời gian và công sức so với việc thủ công thu thập và nhập liệu.
- Sử dụng Selenium (cụ thể là webdriver.Edge) để tự động điều khiển trình duyệt (Edge) nhằm thu thập dữ liệu từ trang web. Bên cạnh đó, bạn kết hợp với BeautifulSoup để phân tích (parse) nội dung HTML sau khi Selenium đã tải trang. Đây là cách tiếp cận “lai” giữa WebDriver và HTML parsing để cào dữ liệu.
- Lưu dữ liệu thô dưới dạng file csv và thực hiện tiền xử lý đầu tiên.

### 2. Thực hiện trích xuất đặc trưng bằng LLM

- Tự động trích xuất các đặc trưng quan trọng từ mô tả bất động sản bằng cách sử dụng API của Anyscale. Các đặc trưng này bao gồm giá, diện tích, địa chỉ, số phòng ngủ, số phòng tắm, số tầng, tiện ích, tên đường, tên phường, tên quận, số mặt tiền, và thông tin về vị trí nhà. Mã nguồn được thiết kế để tăng cường hiệu quả và độ chính xác của quá trình phân tích dữ liệu bất động sản thông qua việc tự động hóa việc trích xuất đặc trưng từ các văn bản mô tả.
- Sử dụng biến môi trường để lấy API base URL và token truy cập Anyscale API. Quá trình chuẩn bị dữ liệu bao gồm tạo và cấu trúc hóa DataFrame chứa mô tả bất động sản để sẵn sàng cho việc trích xuất đặc trưng.
- Đoạn prompt là một chuỗi định dạng dài, được sử dụng để tạo yêu cầu gửi đến API của Anyscale để trích xuất các đặc trưng từ văn bản mô tả bất động sản. Đây là một mẫu minh họa rõ ràng cho API về những gì nhóm muốn trích xuất từ văn bản tiếng Việt. Đoạn prompt này không chỉ giúp cho API hiểu rõ yêu cầu của bạn mà còn cung cấp cho người dùng và nhà phát triển một hình dung rõ ràng về những gì được trích xuất từ dữ liệu đầu vào. Việc sử dụng prompt giúp tăng tính mạch lạc và đúng đắn trong quá trình gửi yêu cầu và xử lý kết quả từ Anyscale API.
- Thiết kế để xử lý lượng lớn dữ liệu, chia thành các batch để xử lý một cách hiệu quả. Kết quả từ mỗi batch được lưu trữ vào một DataFrame và sau đó được xuất ra tệp CSV để dễ dàng quản lý và phân tích sau này.

### 3. Tiền xử lý dữ liệu và Kỹ thuật xây dựng đặc trưng

- Sau khi đã cào được dữ liệu từ website và sử dụng API của Anyscale để lấy thêm 1 vài đặc trưng liên quan đến giá nhà, tiếp đến là thực hiện gom 2 bộ dữ liệu thô thành 1 tập dữ liệu thô duy nhất có đầy đủ đặc trưng cần thiết cho mô hình
- Mục tiêu là hợp nhất hai DataFrame df1 và df2 dựa trên cột Listing ID và xử lý các giá trị thiếu hoặc không hợp lệ trong các cột Price, Area, Bedrooms, và Bathrooms bằng cách sử dụng giá trị từ DataFrame còn lại.
- Trong quá trình xử lý và phân tích dữ liệu bất động sản, việc làm sạch và chuyển đổi dữ liệu là một bước quan trọng để đảm bảo chất lượng và tính chính xác của dữ liệu đầu vào cho mô hình dự đoán. Dưới đây mô tả các bước làm sạch dữ liệu từ hai DataFrame và áp dụng các hàm để xử lý các giá trị thiếu hoặc không hợp lệ trong các cột Price, Area, Bedrooms, Bathrooms, Amenities, Floors, Main road, và Frontages.
- Lấy tọa độ địa lý (kinh độ và vĩ độ) của các quận huyện từ một API không đồng bộ và tạo DataFrame chứa thông tin này
- Loại bỏ các outlier bằng kĩ thuật Interquartile Range (IQR), Kĩ thuật này thường được sử dụng để xác định các điểm dữ liệu không bình thường dựa trên phân phối của dữ liệu. phương pháp này sử dụng phân vị 25% (Q1) và phân vị 75% (Q3) của dữ liệu để tính toán khoảng IQR (Interquartile Range), là sự chênh lệch giữa Q3 và Q1. Sau đó, các giá trị ngoại lai thường được xác định dựa trên khoảng IQR bằng cách xác định các giá trị nằm ngoài phạm vi giới hạn dưới (lower_bound = Q1 - 1.5 * IQR) và giới hạn trên (upper_bound = Q3 + 1.5 * IQR). Các giá trị ngoại lai được xác định dựa trên phương pháp này sau đó được loại bỏ khỏi tập dữ liệu.

### 4. Xây dựng các mô hình dự đoán

- Xây dựng 4 mô hình (bao gồm: Linear Regression Model, Ridge Regression Model, Decision Tree Model và Gradient Boosting Model) và để đánh giá các mô hình trên chúng tôi sử dụng các Chỉ số hồi quy (MSE, MAE) và R².
<img width="398" height="124" alt="Image" src="https://github.com/user-attachments/assets/5f94e6be-378d-456f-b430-cbbc0fa51c1f" />

- **Linear Regression:** Kết quả cho thấy với MSE khá cao cùng với MAE (1834.119809), cho thấy dự đoán trên lệch vẫn khá lớn, ngoài ra chỉ có 44.2% biến thiên của giá nhà được giải thích bằng mô hình. Điều này cho thấy hiệu quả dự đoán vẫn còn thấp.
- **Ridge Regression:** Kết quả gần như tương đương với Linear Regression khi cả ba chỉ số MSE, MAE, R² không có sự thay đổi quá nhiều. Điều này có thể do trong dữ liệu không có nhiều hiện tượng đa cộng tuyến mạnh (chỉ có tương quan giữa Bedrooms và Bathrooms ~ 0.76), nên regularization của Ridge không mang lại nhiều tác dụng. Hơn nữa nó cho thấy overfitting chưa phải vấn đề chính, mà nguyên nhân chủ yếu là mô hình còn quá đơn giản (underfitting).
- **Decision Tree Regressor:** Không cải thiện so với Linear và Ridge Regression với MSE và MAE lại cao hơn, cùng với R² thấp hơn (0.378473). Điều này cho thấy mô hình này không phù hợp cho bài toán yêu cầu khả năng bắt các mối quan hệ phi tuyến.
- **Gradient Boosting Regressor:** Có kết quả tốt nhất trong các mô hình với MSE và MAE thấp nhất và R² cao nhất (0.489566). Mô hình này có khả năng kết hợp nhiều cây quyết định để tạo ra dự đoán tốt hơn, bắt được các mối quan hệ phức tạp hơn trong dữ liệu.

### 5. Kết luận

- So sánh các mô hình:
  - Mô hình tốt nhất: Gradient Boosting Model là mô hình tốt nhất dựa trên các chỉ số đánh giá và so với các mô hình khác, với giá trị R² cao nhất và lỗi (MSE và MAE) thấp nhất.
  - Mô hình kém nhất: Decision Tree Regressor là mô hình kém nhất trong các mô hình được so sánh.
 
- Có rất nhiều yếu tố ảnh hưởng đến giá của một căn nhà trong thực tế nhưng trong bộ dữ liệu đã thu thập được trên web thì chỉ lấy được các yếu tố ảnh hưởng đặc biệt là Diện tích, số phòng ngủ và số phòng tắm ngoài ra các yếu tố khác không có tác động nhiều. Có thể 1 phần là do việc xử lý dữ liệu chưa được phù hợp với bộ dữ liệu này hoặc do cách lấy dữ liệu từ việc cào cho đến sử dụng LLM để phân tích lấy ra các đặc trưng, cũng có thể là do người bán đã đăng thông tin chưa được chính xác dẫn đến việc lấy thông tin nhiễu, sai lệch làm cho không thể xây dưng mô hình với nhiều đặc trưng với độ tương quan mạnh.
- Dự án này nhằm cung cấp một phần những hiểu biết hữu ích về thị trường bất động sản và có thể đóng góp một phần vào việc hỗ trợ các quyết định đầu tư, định giá bất động sản và xây dựng các chiến lược kinh doanh. Tuy nhiên, cần lưu ý rằng mô hình dự đoán chỉ là một công cụ hỗ trợ và không thể thay thế hoàn toàn đánh giá của chuyên gia.
