# TÀI LIỆU ĐẶC TẢ THIẾT KẾ (DESIGN SPECIFICATION)

## HỆ THỐNG AGENT EVALUATION SUMMARIZER (BÀI TOÁN 8.5)

---

## 1. Tổng quan Hệ thống (System Overview)

Hệ thống **Agent Evaluation Summarizer** được thiết kế để giải quyết bài toán xử lý lượng số liệu rải rác và phức tạp sau mỗi lượt chạy đánh giá (bao gồm Golden Experiment và Production Log) trên Evaluation Platform của VSF. Thay vì bắt buộc Kỹ sư Vận hành AI hoặc Product Owner (PO) phải tự khai thác dữ liệu thủ công, hệ thống này sẽ đóng vai trò là một **Trợ lý AI tương tác (Chat Agent + Visualization)**. Agent có khả năng tự động đọc hiểu toàn bộ kết quả run, biên soạn báo cáo phân tích thông minh, phân cụm lỗi và hỗ trợ hội thoại đa lượt để truy vấn đào sâu vào nguyên nhân gốc rễ (Root Cause).

### 1.1 Mục tiêu lõi (Core Objectives)

* 
**Tự động hóa phân tích**: Giảm thiểu thời gian từ lúc có kết quả kiểm thử/giám sát đến khi đưa ra quyết định tối ưu hóa prompt/model.


* 
**Giao diện hội thoại kết hợp trực quan (Conversational + Visual UI)**: Cho phép tương tác đặt câu hỏi bằng ngôn ngữ tự nhiên và trả về phản hồi kèm biểu đồ sinh động.


* 
**Bất khả tri về Schema (Schema-Agnostic)**: Đọc hiểu và tổng hợp được đầu ra của mọi loại metric hiện tại (Skill/Tool/Argument, Multi-turn, Multimodal) lẫn các metric tùy biến tương lai.



---

## 2. Kiến trúc Tổng thể & Luồng dữ liệu (Architecture & Data Flow)

Hệ thống được thiết kế theo mô hình **LLM Agent Workflow** sử dụng cơ chế Routing, State Memory và Dynamic Tool Calling để truy vấn kho dữ liệu phân tích.

```
                    +---------------------------------------+
                    |  Data Ingestion Layer (Langfuse / DB)  |
                    +---------------------------------------+
                                        |
                                        v (Raw Run Output JSON)
+---------------------------------------------------------------------------------------+
| Summarizer Agent Core Engine                                                          |
|                                                                                       |
|   +--------------------------+     +------------------------+     +---------------+   |
|   |  Dynamic Parsing Parser  | --> | Error Clustering Unit  | --> | Report Gen    |   |
|   |  (Agnostic Metric Layer) |     | (Taxonomy Mapping)     |     | (Exec/Tech)   |   |
|   +--------------------------+     +------------------------+     +---------------+   |
|                                                                           |           |
|   +-------------------------------------------------------------------+   |           |
|   | Interaction & Conversational Engine                               |<--+           |
|   |                                                                   |               |
|   |  +------------------+     +-----------------+     +------------+  |               |
|   |  | Context Memory   | <-> | Intent Router   | <-> | UI Planner |  |               |
|   |  +------------------+     +-----------------+     +------------+  |               |
|   +-------------------------------------------------------------------+               |
+---------------------------------------------------------------------------------------+
                                        |
                                        v (Rich Component Payload)
                    +---------------------------------------+
                    | UI Layer (Text, Tables, Charts, Link) |
                    +---------------------------------------+

```

### Các thành phần chính:

1. **Dynamic Parsing Layer**: Đọc cấu trúc JSON thô từ `sample.md` (bao gồm `score`, `reasoning`, `llm_step_usage`) để chuẩn hóa thành mô hình dữ liệu tập trung.
2. 
**Error Clustering Unit**: Phân cụm các ca thất bại theo Taxonomy lỗi đã được định nghĩa ở các bài toán thành phần (lỗi chọn skill, lỗi chọn tool, lỗi đối số, lỗi context retention, lỗi grounded, v.v.).


3. 
**Context Memory**: Bộ nhớ lưu trữ trạng thái phiên hội thoại, đảm bảo khi user hỏi các câu kế tiếp ("Tại sao nó fail?", "Show case cụ thể"), Agent hiểu rõ ngữ cảnh của run đang được chọn.


4. 
**UI Planner**: Chuyển đổi phản hồi của LLM thành một Payload kết hợp giữa văn bản markdown và các UI Block (Biểu đồ, bảng dữ liệu, Deep-link).



---

## 3. Thiết kế Data Model nội bộ (Internal Data Model)

Agent sẽ map toàn bộ dữ liệu đầu vào thành một schema phân tích toàn diện. Cấu trúc này tương đồng với file mẫu `output.md` được cung cấp:

```json
{
  "report_metadata": {
    "run_id": "string",
    "agent_name": "string",
    "evaluation_timestamp": "ISO-8601",
    "environment": "experiment | production_log_eval",
    "target_version": "string"
  },
  "executive_view": {
    "summary_metrics": {
      "pass_rate_global": "float",
      "total_evaluated_samples": "int",
      "global_status": "PASS | FAIL",
      "total_run_cost_usd": "float"
    },
    "breakdown_by_skill_group": [
      {
        "skill_group_name": "string",
        "pass_rate": "float",
        "total_cases": "int",
        "status": "HEALTHY | UNHEALTHY"
      }
    ],
    "top_actionable_insights": [
      {
        "priority_rank": "int",
        "metric_id": "string",
        "metric_display_name": "string",
        "impact_vs_effort": "string",
        "recommended_action": "string",
        "business_optimization_tip": "string"
      }
    ]
  },
  "technical_view": {
    "cross_run_comparison": {
      "compared_with_baseline_run": "string",
      "improved_cases_count": "int",
      "unchanged_cases_count": "int",
      "regressed_cases_count": "int",
      "regression_alert": "boolean",
      "regression_details": []
    },
    "fail_patterns_clustering": [
      {
        "cluster_id": "string",
        "error_code": "string",
        "error_description": "string",
        "sample_count": "int",
        "percentage_of_total": "float",
        "representative_samples": [
          {
            "trace_id": "string",
            "user_query": "string",
            "evaluator_reasoning": "string"
          }
        ]
      }
    ],
    "cost_and_performance_analytics": {
      "total_tokens_consumed": "int",
      "tokens_breakdown": { "input_tokens": "int", "output_tokens": "int" },
      "pipeline_steps_telemetry": [
        {
          "step_index": "int",
          "step_name": "string",
          "provider": "string",
          "model": "string",
          "token_usage": { "in": "int", "out": "int" }
        }
      ],
      "cache_efficiency": { "cache_hit_rate": "float", "saved_cost_usd": "float" }
    }
  }
}

```

---

## 4. Kế hoạch Triển khai Chi tiết theo từng Sprint (Sprint Roadmap)

Dự án được chia thành **5 Sprint** (Mỗi Sprint kéo dài 2 tuần), đảm bảo tính tiệm tiến từ hạ tầng dữ liệu đến tính năng Agent thông minh nâng cao.

### SPRINT 1: Core Parsing Engine & Data Ingestion

**Mục tiêu**: Xây dựng tầng Ingestion để đọc, phân tích và chuẩn hóa mọi cấu trúc JSON kết quả đầu ra từ các bộ Evaluator (Bài toán 8.1 - 8.3).

* **Tính năng chi tiết**:
* Phát triển các module parser bất khả tri (Schema-Agnostic Parser) có khả năng tự động trích xuất các trường dữ liệu cốt lõi như `score`, `reasoning`, và `llm_step_usage` từ bất kỳ định dạng JSON thô nào của Evaluator gửi về.
* Thiết kế công cụ kết nối (Data Connector) để kéo trace logs lịch sử chạy đánh giá từ cơ sở dữ liệu lưu trữ tập trung hoặc từ hệ thống giám sát Langfuse dựa theo tham số `run_id`.


* Tích hợp hệ thống phân tích chi phí và token (Cost & Telemetry Metric Aggregator) để tính toán tổng số tiền tiêu thụ, lượng token in/out, thống kê hiệu suất cache của từng bước thực thi trong pipeline kiểm thử.


* **Đầu ra kỹ thuật (Deliverables)**:
* Bộ thư viện Core Parsing Engine (Python/TypeScript).
* Hệ thống API nội bộ `/api/v1/eval-runs/{run_id}/ingest` trả về dữ liệu cấu trúc hóa đã qua tinh lọc.



### SPRINT 2: Analytical Reporter & Clustering Engine

**Mục tiêu**: Phát triển công cụ tự động tạo báo cáo tĩnh (Tự động sinh khi mở báo cáo) bao gồm hai chế độ hiển thị Executive và Technical, đồng thời triển khai thuật toán phân cụm lỗi.

* **Tính năng chi tiết**:
* 
**Phân cụm lỗi (Error Clustering Engine)**: Áp dụng LLM kết hợp thuật toán phân cụm văn bản dựa trên embedding để gộp các ca kiểm thử thất bại (Fail cases) vào các nhóm lỗi chung, tự động gán nhãn dựa trên Taxonomy lỗi (như chọn sai skill, truyền thiếu tham số, đọc sai ảnh mờ, mất ngữ cảnh hội thoại).


* 
**Sinh báo cáo tự động (Auto-Report Generator)**: Viết prompt template chuyên biệt cho LLM để tạo lập tức hai góc nhìn khi mở report:


* 
*Executive View*: Tập trung vào tỷ lệ Pass Rate tổng thể, phân tích chi phí, trạng thái sức khỏe của từng nhóm sản phẩm AI và các đề xuất hành động ưu tiên dựa trên ma trận Impact × Effort.


* *Technical View*: Đi sâu vào chi tiết so sánh chéo (Cross-run regression analysis) với bản baseline trước đó, danh sách phân cụm lỗi kèm bằng chứng và dữ liệu đo đạc tài nguyên.


* 
**Chẩn đoán liên chuỗi (Root Cause Diagnosis)**: Huấn luyện tư duy cho hệ thống để nhận biết liên kết lỗi chéo (ví dụ: phát hiện lỗi rò rỉ tham số ở tầng argument thực chất xuất phát từ việc chọn sai skill hệ thống ngay ở bước routing đầu tiên).




* **Đầu ra kỹ thuật (Deliverables)**:
* Module `ErrorClusteringService` và `ReportGenerationService`.
* Hệ thống sinh cấu trúc JSON hoàn chỉnh cho view tổng quan (giống cấu trúc mẫu `output.md`).



### SPRINT 3: Chat Agent Orchestration & State Management

**Mục tiêu**: Xây dựng kiến trúc Runtime của Chat Agent, quản lý bộ nhớ hội thoại đa lượt và cơ chế chuyển đổi từ ngôn ngữ tự nhiên thành mã truy vấn dữ liệu.

* **Tính năng chi tiết**:
* 
**Quản lý bộ nhớ ngữ cảnh (Agent Memory Manager)**: Triển khai Session-based Memory giúp Agent giữ vững ngữ cảnh của toàn bộ run hiện hành, cho phép người dùng hỏi các câu kế thừa dạng đại từ chỉ định mà không phải lặp lại thông tin.


* 
**Bộ điều hướng ý định (Intent Router & Query Translator)**: Phát triển bộ phân tích cú pháp ngôn ngữ tự nhiên để biên dịch các câu hỏi phức tạp của PO/Kỹ sư (ví dụ: *"Show 10 fail case tệ nhất skill booking"*, *"4 run gần nhất metric nào trend xấu?"*) thành mã truy vấn dữ liệu có cấu trúc (SQL / JSON Filters) để kéo thông tin chính xác từ kho lưu trữ.


* 
**Kết xuất bằng chứng (Evidence Linking)**: Thiết kế cơ chế tự động mapping kết quả trả lời bằng text với ID của các trường dữ liệu thực tế, chuẩn bị dữ liệu đính kèm liên kết sâu (Deep-link) trực tiếp dẫn tới từng ca lỗi cụ thể để phục vụ việc debug.




* **Đầu ra kỹ thuật (Deliverables)**:
* Agent Runtime Orchestrator tích hợp với LangChain/Semantic Kernel.
* Engine xử lý câu hỏi và trả về chuỗi cấu trúc dữ liệu thô phục vụ hiển thị.



### SPRINT 4: Visualization & UI Component Rendering Integration

**Mục tiêu**: Xây dựng giao diện hiển thị kết hợp (Hybrid UI) biến Payload cấu trúc từ Agent thành các thành phần đồ họa tương tác trực quan.

* **Tính năng chi tiết**:
* 
**Thư viện UI Block linh hoạt**: Xây dựng các UI component phía Frontend (React/Vue) có thể render động từ mã Payload do Agent trả về:


* 
*Biểu đồ cột (Bar chart)* thể hiện trực quan tỷ lệ Pass Rate theo từng Skill cụ thể.


* 
*Đồ thị xu hướng (Line chart)* theo dõi biến động chất lượng qua chuỗi các lượt chạy kiểm thử khác nhau.


* 
*Bản đồ nhiệt (Heatmap)* thể hiện vùng mật độ tập trung lỗi nghiêm trọng.




* 
**Tích hợp bảng biểu & Clickable Link**: Thiết kế các bảng dữ liệu kỹ thuật có khả năng lọc, phân trang, hỗ trợ nhấp chuột trực tiếp vào Trace ID để mở tab chi tiết xem log hội thoại hoặc file ảnh/PDF gốc của người dùng.


* 
**Đồng bộ hóa Trạng thái Text-Visual**: Đảm bảo Agent có khả năng sinh câu thoại dẫn dắt song song với việc kích hoạt render đúng mã biểu đồ tương ứng ngay trong luồng chat trực tuyến.




* **Đầu ra kỹ thuật (Deliverables)**:
* Hệ thống Design System nội bộ gồm các Component: `<AgentChart />`, `<AgentTable />`, `<TraceDeepLink />`.
* Giao diện khung Chat Web hoàn thiện cho phép hiển thị nội dung trực quan đan xen văn bản.



### SPRINT 5: Cross-Source Synthesis Engine & Production Discovery

**Mục tiêu**: Triển khai tính năng tổng hợp dữ liệu chéo nâng cao giữa hai môi trường Experiment (Pre-release) và Production Log (Post-release).

* **Tính năng chi tiết**:
* 
**Khai phá khoảng trống dữ liệu (Coverage Gap Analysis)**: Lập trình logic cho phép Agent xử lý câu hỏi nâng cao: *"Production tuần này có lỗi gì golden dataset chưa cover?"*. Agent sẽ tự động đối chiếu các mô hình phân cụm lỗi thu được ở production với tập dữ liệu chuẩn mực của Golden Dataset để chỉ ra các khoảng trống kiểm thử.


* 
**Gợi ý làm giàu dữ liệu (Dataset Promotion Workflow)**: Thiết kế tính năng tự động đề xuất, trích lọc các ca biên nguy hiểm (edge case) hoặc các lỗi thực tế điển hình ngoài production để đẩy ngược lại vào quy trình Curate và nâng cấp phiên bản cho Golden Dataset.


* 
**Giám sát Anomaly & Version Drift**: Xây dựng thuật toán phân tích xu hướng chạy dài hạn nhằm phát hiện xem phiên bản Agent mới triển khai có gặp hiện tượng hồi quy chất lượng âm thầm theo thời gian hay không.




* **Đầu ra kỹ thuật (Deliverables)**:
* Engine `CrossSourceSynthesisService`.
* Bảng khuyến nghị tối ưu hóa Golden Dataset tích hợp trực tiếp trong cửa sổ chat của Agent.


* Đóng gói toàn diện giải pháp, hoàn thiện kiểm thử hệ thống (End-to-End Testing).



---

## 5. Tiêu chí Đánh giá Chất lượng Agent (Evaluation & Observability)

Để đảm bảo hệ thống Summarizer hoạt động ổn định, chính xác và không đưa ra thông tin sai lệch, các chỉ số sau sẽ được giám sát chặt chẽ:

| Nhóm Chỉ số | Metric Đo lường | Phương pháp Kiểm tra | Tiêu chuẩn Đạt (SLA) |
| --- | --- | --- | --- |
| **Độ chính xác dữ liệu** | Data Extraction Accuracy | So sánh thông tin số liệu Agent nói trong hội thoại với dữ liệu JSON gốc của Run | **≥ 99.5%** (Tuyệt đối không sai lệch số liệu) |
| **Phân cụm lỗi** | Clustering F1-Score | Đối chiếu kết quả phân cụm lỗi tự động của Agent với nhãn Taxonomy do chuyên gia kiểm định thủ công| **≥ 90%** |
| **Khả năng giữ ngữ cảnh** | Context Retention Rate | Chạy chuỗi câu hỏi thử nghiệm đa lượt (3-5 turns) liên quan đến cùng một run thất bại | **≥ 95%** phản hồi đúng ngữ cảnh |
| **Độ chính xác đề xuất** | Actionable Insight Relevance | Khảo sát ý kiến đội ngũ PO/Dev về tính hữu ích và chính xác của đề xuất sửa đổi prompt/model | **≥ 85%** đánh giá hữu ích |

---
