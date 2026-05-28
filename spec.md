Dưới đây là toàn bộ tài liệu đặc tả thiết kế (Design Specification) hoàn chỉnh của hệ thống **Agent Evaluation Summarizer (Bài toán 8.5)**. Danh mục các **Skill** và **Tool** đã được bổ sung và lồng ghép một cách đồng nhất vào đúng cấu trúc luồng kiến trúc, mô hình dữ liệu và lộ trình phát triển theo từng Sprint của tài liệu gốc.

---

# TÀI LIỆU ĐẶC TẢ THIẾT KẾ (DESIGN SPECIFICATION)

## HỆ THỐNG AGENT EVALUATION SUMMARIZER (BÀI TOÁN 8.5)

---

## 1. Tổng quan Hệ thống (System Overview)

Hệ thống **Agent Evaluation Summarizer** được xây dựng nhằm giải quyết bài toán xử lý lượng số liệu rải rác, phức tạp sau mỗi lượt chạy đánh giá (Golden Experiment hoặc Production Log) trên nền tảng Evaluation Platform của VSF. Thay vì bắt buộc Kỹ sư Vận hành AI hoặc Product Owner (PO) phải tự khai thác dữ liệu thủ công, hệ thống này đóng vai trò là một **Trợ lý AI tương tác (Chat Agent kết hợp Visualize)**. Agent có khả năng tự động đọc hiểu toàn bộ kết quả run, biên soạn báo cáo phân tích thông minh, phân cụm lỗi và hỗ trợ hội thoại đa lượt để truy vấn đào sâu vào nguyên nhân gốc rễ (Root Cause).

### 1.1 Mục tiêu lõi (Core Objectives)

* 
**Tự động hóa phân tích**: Giảm thiểu thời gian từ lúc có kết quả kiểm thử/giám sát đến khi đưa ra quyết định tối ưu hóa prompt/model.


* 
**Giao diện hội thoại kết hợp trực quan (Conversational + Visual UI)**: Cho phép tương tác đặt câu hỏi bằng ngôn ngữ tự nhiên và trả về phản hồi kèm biểu đồ, bảng biểu sinh động.


* 
**Bất khả tri về Schema (Schema-Agnostic)**: Đọc hiểu và tổng hợp được đầu ra của mọi loại metric hiện tại (Skill/Tool/Argument, Multi-turn, Multimodal) lẫn các metric tùy biến tương lai.



---

## 2. Kiến trúc Tổng thể & Nguyên lý Vận hành Agent

Hệ thống vận hành theo mô hình **Agentic Workflow** sử dụng cơ chế Routing, State Memory và các công cụ thực thi mã lệnh (Tools) được điều phối thông qua các năng lực nhận thức cấp cao (Skills).

### 2.1 Thành phần lõi trong kiến trúc:

* 
**Tầng Nhận thức (Skills)**: Các module logic độc lập, chịu trách nhiệm phân tích ý định người dùng (User Intent) từ hội thoại để lên kế hoạch xử lý, quyết định gọi Tool nào và khi nào cần trả về kết quả.


* 
**Tầng Thực thi (Tools)**: Các hàm mã lệnh phần mềm (Python/TypeScript Functions), API truy vấn Database hoặc các lượt gọi LLM chuyên biệt (như LLM-as-a-judge/Classifier) thực hiện các tác vụ kỹ thuật chuyên sâu.


* 
**Bộ nhớ ngữ cảnh (Context Memory)**: Lưu trữ trạng thái phiên hội thoại, giúp Agent giữ vững ngữ cảnh của toàn bộ run hiện hành khi người dùng đặt câu hỏi nối tiếp.



---

## 3. Chi tiết Danh mục Bộ Skills & Tools của Agent

### 3.1. Các SKILLS (Năng lực Nhận thức của Agent)

* 
**`Skill_Generate_Run_Summary` (Tổng hợp báo cáo tự động)**: Tự động kích hoạt ngay khi người dùng mở giao diện báo cáo của một Run ID. Skill này sẽ điều phối các Tool để trích xuất tỷ lệ đạt (Pass rate), chi phí, phân cụm lỗi và trả về cấu trúc dữ liệu cho hai chế độ hiển thị: Executive View và Technical View.


* 
**`Skill_Diagnose_Failure_Patterns` (Chẩn đoán nguyên nhân gốc rễ)**: Chịu trách nhiệm phân tích các chuỗi lỗi liên đới. Ví dụ: Đưa ra giả thuyết chẩn đoán lỗi truyền sai đối số (`argument_correctness`) thực chất xuất phát từ việc chọn sai skill hệ thống (`skill_selection_accuracy`) ngay ở bước routing đầu tiên.


* 
**`Skill_Drill_Down_Query` (Hỏi đáp truy vấn ca lỗi chi tiết)**: Tiếp nhận câu hỏi tự do của người dùng (Ví dụ: *"Show 10 fail case tệ nhất skill booking"*), dịch ý định thành tham số truy vấn để trích xuất chính xác các bản ghi mẫu kèm Deep-link dẫn tới Langfuse.


* 
**`Skill_Analyze_Cross_Run_Trends` (Phân tích xu hướng dài hạn)**: Thực hiện so sánh hiệu năng giữa các lượt chạy để kiểm tra lỗi hồi quy (Regression analysis) hoặc phân tích xu hướng biến động chất lượng/chi phí qua chuỗi nhiều lượt chạy.


* 
**`Skill_Discover_Production_Gaps` (Phân tích khoảng trống dữ liệu thực tế)**: Xử lý câu hỏi nâng cao dạng: *"Production tuần này có lỗi gì golden dataset chưa cover?"*. Skill này tự động đối chiếu mô hình phân cụm lỗi ngoài Production với tập dữ liệu chuẩn mực để gợi ý bổ sung ca biên (Edge cases) vào Golden Dataset.



### 3.2. Các TOOLS (Công cụ Thực thi Mã lệnh/API)

* **`Tool_Fetch_Evaluation_Raw_Data`**: API kết nối vào kho lưu trữ (hoặc Langfuse) để tải toàn bộ mảng dữ liệu JSON thô của lượt chạy kiểm thử, tự động bóc tách các trường cốt lõi (`score`, `reasoning`, `llm_step_usage`).
* 
**`Tool_LLM_Error_Classifier_Clustering`**: Gọi một LLM chuyên biệt đóng vai trò Text Classifier nhằm đọc qua hàng loạt lý do thất bại thô, tiến hành gom nhóm ngữ nghĩa và gắn mã lỗi chuẩn hóa theo đúng sơ đồ phân loại (Taxonomy) lỗi.


* **`Tool_Aggregate_Cost_Performance`**: Hàm toán học tính toán số liệu tài nguyên cứng. Duyệt qua mảng dữ liệu `llm_step_usage` của từng ca để tính tổng lượng token tiêu thụ, quy đổi ra chi phí USD, và đo lường tỷ lệ tiết kiệm nhờ bộ nhớ đệm (Cache hit rate).
* 
**`Tool_Query_Analytical_Engine`**: Nhận một đối tượng lọc cấu trúc (JSON Filter Object) do LLM biên dịch từ câu thoại của user, thực thi câu lệnh truy vấn trực tiếp xuống Database phân tích để rút ra danh sách các bản ghi mẫu chính xác.


* 
**`Tool_Diff_Metrics_Engine`**: Thực hiện phép toán so sánh (Diff) giữa hai ma trận điểm số của hai Run khác nhau để trích xuất các chỉ số biến động (Delta) hiệu năng và lỗi hồi quy.


* 
**`Tool_UI_Component_Formatter`**: Định dạng dữ liệu thô thu được từ các Tool tính toán thành cấu trúc mã Payload đồ họa tiêu chuẩn (JSON UI Component) phục vụ hiển thị (Biểu đồ cột, đồ thị đường xu hướng, bảng dữ liệu có phân trang, Deep-link Chips).



---

## 4. Thiết kế Data Model nội bộ (Internal Data Model)

Agent sẽ chuẩn hóa toàn bộ dữ liệu đầu vào và kết quả đầu ra của các bộ Tools thành một schema cấu trúc tập trung (tương đồng với định dạng mẫu `output.md`):

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

## 5. Kế hoạch Triển khai Chi tiết theo từng Sprint (Sprint Roadmap)

### SPRINT 1: Core Parsing Engine & Data Ingestion

**Mục tiêu**: Xây dựng hạ tầng kết nối dữ liệu thô và công cụ đo lường tài nguyên, chi phí vận hành.

* **Các cấu phần phát triển**:
* Phát triển **`Tool_Fetch_Evaluation_Raw_Data`**: Module parser bất khả tri (Schema-Agnostic Parser) có khả năng tự động trích xuất các trường dữ liệu cốt lõi như `score`, `reasoning`, và `llm_step_usage` từ cấu trúc JSON thô của Evaluator gửi về.


* Phát triển **`Tool_Aggregate_Cost_Performance`**: Tích hợp hệ thống phân tích chi phí và token, duyệt qua mảng dữ liệu `llm_step_usage` để thống kê lượng token in/out, quy đổi chi phí USD và ghi nhận hiệu suất cache.




* **Đầu ra kỹ thuật (Deliverables)**:
* Mã nguồn hoàn chỉnh của hai công cụ nền tảng (`Tool_Fetch_Evaluation_Raw_Data`, `Tool_Aggregate_Cost_Performance`).
* Hệ thống API nội bộ `/api/v1/eval-runs/{run_id}/ingest` trả về dữ liệu cấu trúc hóa đã qua tinh lọc tài nguyên.



### SPRINT 2: Analytical Reporter & Clustering Engine

**Mục tiêu**: Ứng dụng LLM phân cụm lỗi tự động và hoàn thiện năng lực sinh báo cáo tổng quan khi mở dashboard.

* **Các cấu phần phát triển**:
* Phát triển **`Tool_LLM_Error_Classifier_Clustering`**: Tích hợp Prompt chuyên biệt gọi lên Azure OpenAI (Mô hình GPT-5.2 hoặc tương đương) đóng vai trò bộ phân cụm văn bản dựa trên ngữ nghĩa, tự động gán nhãn dựa trên Taxonomy lỗi hệ thống.


* Phát triển **`Skill_Generate_Run_Summary`**: Điều phối các công cụ đã xây dựng ở Sprint 1 và 2 để tự động lắp ghép dữ liệu số liệu cứng và danh sách các cụm lỗi, viết phần tóm tắt cho hai góc nhìn: *Executive View* (Ma trận Impact × Effort) và *Technical View* (Giả thuyết nguyên nhân gốc rễ).




* **Đầu ra kỹ thuật (Deliverables)**:
* Module logic `ErrorClusteringService` và `ReportGenerationService`.
* Hệ thống tự động xuất cấu trúc JSON hoàn chỉnh cho view tổng quan tĩnh ngay khi mở báo cáo.





### SPRINT 3: Chat Agent Orchestration & State Management

**Mục tiêu**: Xây dựng nền tảng tương tác thời gian thực, quản lý bộ nhớ hội thoại đa lượt và cơ chế truy vấn động.

* **Các cấu phần phát triển**:
* Phát triển **Bộ nhớ ngữ cảnh (Context Memory)**: Thiết kế Session-based Memory giúp Agent giữ vững ngữ cảnh của toàn bộ run hiện hành, cho phép người dùng hỏi các câu hỏi kế thừa đa lượt.


* Phát triển **`Tool_Query_Analytical_Engine`**: Bộ biên dịch ngôn ngữ tự nhiên thành các tham số truy vấn dữ liệu có cấu trúc (SQL Commands hoặc JSON Filters) để truy xuất thông tin từ Database.


* Triển khai hoàn thiện **`Skill_Drill_Down_Query`** và **`Skill_Diagnose_Failure_Patterns`**: Cho phép Agent tiếp nhận các câu thoại tự do, thực thi truy vấn động để trả về chính xác danh sách các ca lỗi cụ thể hoặc liên kết chéo các tầng lỗi để tìm root cause.




* **Đầu ra kỹ thuật (Deliverables)**:
* Khung Agent Runtime Orchestrator quản lý hội thoại đa lượt ổn định.
* Động cơ chat phản hồi đúng danh sách mẫu ca lỗi kèm liên kết ID khi được yêu cầu bằng ngôn ngữ tự nhiên.





### SPRINT 4: Visualization & UI Component Rendering Integration

**Mục tiêu**: Xây dựng giao diện hiển thị kết hợp (Hybrid UI) biến Payload cấu trúc từ Agent thành các thành phần đồ họa tương tác trực quan ngay trong luồng chat.

* **Các cấu phần phát triển**:
* Phát triển **`Tool_UI_Component_Formatter`**: Chịu trách nhiệm chuyển đổi cấu trúc văn bản hoặc mảng dữ liệu thô thu được từ tầng Database thành mã Payload JSON quy chuẩn để vẽ đồ họa.


* Tích hợp các UI Blocks linh hoạt phía Frontend (React/Vue) để render động từ mã Payload do Agent trả về: Biểu đồ cột (`Bar Chart` thể hiện Pass Rate theo skill), đồ thị xu hướng (`Line Chart`), bảng dữ liệu lọc có phân trang (`Interactive Table`), và thẻ chứa liên kết sâu (`Deep-link Chips`) dẫn trực tiếp sang Langfuse.




* **Đầu ra kỹ thuật (Deliverables)**:
* Thư viện Component hiển thị phía Frontend (`<AgentChart />`, `<AgentTable />`, `<TraceDeepLink />`).
* Giao diện khung Chat Web hoàn thiện: Người dùng chat yêu cầu, Agent vừa trả lời bằng văn bản vừa kích hoạt hiển thị đúng mô hình biểu đồ tương ứng.





### SPRINT 5: Cross-Source Synthesis Engine & Production Discovery

**Mục tiêu**: Triển khai các tính năng nâng cao về so sánh chéo, kiểm thử lỗi hồi quy dài hạn và đồng bộ khoảng trống dữ liệu thực tế.

* **Các cấu phần phát triển**:
* Phát triển **`Tool_Diff_Metrics_Engine`**: Thực hiện thuật toán so sánh delta chỉ số chất lượng giữa các phiên bản chạy.


* Phát triển **`Skill_Analyze_Cross_Run_Trends`**: Đối chiếu điểm số của phiên bản hiện tại với phiên bản nền tảng (Baseline) nhằm phát hiện sớm hiện tượng hồi quy chất lượng âm thầm theo thời gian.


* Phát triển **`Skill_Discover_Production_Gaps`**: Quét cấu trúc dữ liệu lỗi ngoài Production Log, đối chiếu mô hình cụm lỗi với độ phủ của Golden Dataset hiện tại để chỉ ra khoảng trống kiểm thử, đồng thời đưa ra khuyến nghị trích xuất ca biên để làm giàu bộ dữ liệu chuẩn mực.




* **Đầu ra kỹ thuật (Deliverables)**:
* Module logic nâng cao `CrossSourceSynthesisService`.
* Đóng gói toàn diện giải pháp, hoàn thiện kiểm thử hệ thống (End-to-End Testing) và bàn giao nghiệm thu.



---

## 6. Tiêu chí Đánh giá Chất lượng Agent (Evaluation & Observability)

Để đảm bảo hệ thống Summarizer hoạt động ổn định, chính xác và không đưa ra thông tin sai lệch (hallucination), các chỉ số sau sẽ được giám sát chặt chẽ:

| Nhóm Chỉ số | Metric Đo lường | Phương pháp Kiểm tra | Tiêu chuẩn Đạt (SLA) |
| --- | --- | --- | --- |
| **Độ chính xác dữ liệu** | Data Extraction Accuracy | So sánh thông tin số liệu Agent nói trong hội thoại với dữ liệu JSON gốc của Run | **≥ 99.5%** (Tuyệt đối không sai lệch số liệu) |
| **Phân cụm lỗi** | Clustering F1-Score | Đối chiếu kết quả phân cụm lỗi tự động của `Tool_LLM_Error_Classifier_Clustering` với nhãn do chuyên gia kiểm định thủ công 

 | **≥ 90%** |
| **Khả năng giữ ngữ cảnh** | Context Retention Rate | Chạy chuỗi câu hỏi thử nghiệm đa lượt (3-5 turns) liên quan đến cùng một run thất bại 

 | **≥ 95%** phản hồi đúng ngữ cảnh |
| **Độ chính xác đề xuất** | Actionable Insight Relevance | Khảo sát ý kiến đội ngũ PO/Dev về tính hữu ích và chính xác của đề xuất sửa đổi prompt/model 

 | **≥ 85%** đánh giá hữu ích |
