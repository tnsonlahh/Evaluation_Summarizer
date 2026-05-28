* **Developer A (Core Backend & LLM Specialist):** Phụ trách cấu trúc dữ liệu thô, prompt engineering cho LLM, thuật toán phân cụm lỗi và kiến trúc cốt lõi của Agent.
* **Developer B (Fullstack & UI/Integration Specialist):** Phụ trách pipeline tính toán số liệu cứng, xây dựng API endpoints, hệ thống chuyển đổi câu hỏi thành truy vấn dữ liệu (Text-to-Query) và phát triển giao diện hiển thị đồ họa (Visual UI Component).

---

## SPRINT 1: Core Parsing Engine & Data Ingestion

### Mục tiêu chung

Xây dựng hạ tầng kết nối dữ liệu thô và công cụ đo lường tài nguyên, chi phí vận hành hệ thống.

### Phân chia việc chi tiết

* **Developer A (Core Backend):**
* Xây dựng module **`Tool_Fetch_Evaluation_Raw_Data`** để tải toàn bộ mảng dữ liệu JSON thô của lượt chạy kiểm thử từ hệ thống lưu trữ tập trung hoặc Langfuse.
* Lập trình logic xử lý bất khả tri về cấu trúc (Schema-Agnostic Parser) để tự động trích xuất các trường dữ liệu cốt lõi như `score`, `reasoning`, và `llm_step_usage` từ bất kỳ định dạng Evaluator nào gửi về.


* **Developer B (Fullstack):**
* Xây dựng module **`Tool_Aggregate_Cost_Performance`** để duyệt qua mảng dữ liệu `llm_step_usage` của từng ca đánh giá nhằm tính toán tổng số lượng token tiêu thụ và quy đổi ra chi phí USD thực tế.
* Thiết kế logic đo lường hiệu suất hoạt động của bộ nhớ đệm (Cache hit rate).
* Phát triển và expose hệ thống API nội bộ đầu vào: `/api/v1/eval-runs/{run_id}/ingest`.



---

## SPRINT 2: Analytical Reporter & Clustering Engine

### Mục tiêu chung

Ứng dụng LLM phân cụm lỗi tự động và hoàn thiện năng lực tự sinh báo cáo tĩnh khi mở dashboard.

### Phân chia việc chi tiết

* **Developer A (LLM Specialist):**
* Xây dựng module **`Tool_LLM_Error_Classifier_Clustering`**, thực hiện Prompt Engineering chuyên biệt gọi lên Azure OpenAI (Mô hình GPT-5.2) để gom nhóm các ca thất bại theo ngữ nghĩa.
* Thiết kế cấu trúc phân loại (Taxonomy mapping) để tự động gán nhãn chuẩn hóa cho các cụm lỗi (ví dụ: lỗi chọn sai skill, truyền thiếu tham số).




* **Developer B (Fullstack):**
* Xây dựng năng lực **`Skill_Generate_Run_Summary`** nhằm điều phối các Tool từ Sprint 1 và Sprint 2.
* Lập trình logic kết hợp tự động số liệu chi phí và danh sách cụm lỗi để sinh ra file cấu trúc JSON Báo cáo hoàn chỉnh gồm hai chế độ hiển thị: *Executive View* (Ma trận Impact × Effort) và *Technical View* (Giả thuyết nguyên nhân gốc rễ).



---

## SPRINT 3: Chat Agent Orchestration & State Management

### Mục tiêu chung

Xây dựng nền tảng tương tác thời gian thực, quản lý bộ nhớ hội thoại đa lượt và cơ chế truy vấn động.

### Phân chia việc chi tiết

* **Developer A (Core Backend & LLM):**
* Thiết lập khung Agent Runtime Orchestrator (sử dụng các thư viện như LangChain hoặc Semantic Kernel).


* Triển khai bộ quản lý **Bộ nhớ ngữ cảnh (Context Memory)** theo cơ chế Session-based Memory giúp Agent giữ vững ngữ cảnh của toàn bộ run hiện hành qua nhiều lượt trò chuyện.


* Xây dựng năng lực **`Skill_Diagnose_Failure_Patterns`** để lập luận liên kết chéo các tầng lỗi phục vụ việc chẩn đoán lý do sụt giảm chất lượng của hệ thống.




* **Developer B (Fullstack):**
* Xây dựng công cụ **`Tool_Query_Analytical_Engine`** làm nhiệm vụ tiếp nhận câu hỏi tự nhiên của người dùng, biên dịch thành các tham số lọc cấu trúc cứng hoặc câu lệnh truy vấn dữ liệu xuống Database.


* Triển khai năng lực **`Skill_Drill_Down_Query`** để tiếp nhận các tham số lọc này, thực thi trích xuất chính xác danh sách các ca lỗi cụ thể kèm Deep-link tương ứng.





---

## SPRINT 4: Visualization & UI Component Rendering Integration

### Mục tiêu chung

Xây dựng giao diện hiển thị kết hợp (Hybrid UI) biến Payload cấu trúc từ Agent thành các thành phần đồ họa tương tác trực quan ngay trong luồng chat.

### Phân chia việc chi tiết

* **Developer A (Core Backend):**
* Phát triển công cụ **`Tool_UI_Component_Formatter`** chịu trách nhiệm nhận dữ liệu thô từ tầng Database phân tích, định dạng chúng thành cấu trúc mã Payload JSON UI chuẩn hóa.


* Đồng bộ hóa luồng text của LLM để đảm bảo chỉ thị render biểu đồ được chèn chính xác vào luồng câu thoại phản hồi.




* **Developer B (Frontend):**
* Xây dựng bộ thư viện UI Blocks linh hoạt phía Frontend (React/Vue) bao gồm các component tương tác động: `<AgentChart />` (Biểu đồ cột biểu diễn pass rate theo skill, đồ thị đường xu hướng), `<AgentTable />` (Bảng dữ liệu lỗi có khả năng phân trang, tìm kiếm), và `<TraceDeepLink />`.
* Tích hợp các UI Blocks này vào cửa sổ chat chính, đảm bảo giao diện render mượt mà khi nhận Payload chỉ thị từ Agent.





---

## SPRINT 5: Cross-Source Synthesis Engine & Production Discovery

### Mục tiêu chung

Triển khai các tính năng nâng cao về so sánh chéo, kiểm thử lỗi hồi quy dài hạn và đồng bộ khoảng trống dữ liệu thực tế.

### Phân chia việc chi tiết

* **Developer A (LLM Specialist):**
* Xây dựng công cụ toán học **`Tool_Diff_Metrics_Engine`** để tính toán biến động chỉ số chất lượng giữa hai lượt chạy.


* Triển khai năng lực nâng cao **`Skill_Discover_Production_Gaps`** thực hiện quét Production Log, đối chiếu mô hình cụm lỗi thực tế với độ phủ bài thử nghiệm của Golden Dataset để tìm khoảng trống dữ liệu và đưa ra khuyến nghị làm giàu tập kiểm thử.




* **Developer B (Fullstack):**
* Triển khai năng lực **`Skill_Analyze_Cross_Run_Trends`** dựa trên `Tool_Diff_Metrics_Engine` của Dev A để vẽ biểu đồ theo dõi xu hướng hồi quy chất lượng dài hạn của các phiên bản Agent.


* Chịu trách nhiệm viết kịch bản kiểm thử tích hợp toàn diện hệ thống (End-to-End Testing), tối ưu hóa luồng gọi Tool và đóng gói hoàn thiện giải pháp để bàn giao nghiệm thu.