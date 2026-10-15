# QA Report: Sprint 2

**Project:** Automated Fruit Quality Grading and Freshness Classification System

**Course:** CPE178P Foundations of AI

**Prepared by:** Richard Von Sopongco (Quality Assurance Support)

**Application tested:** Containerized deployment orchestrating FastAPI backend and Streamlit frontend via Docker Compose

---

## 1. Scope

This report covers (a) verification of the group's single-command Docker deployment setup (`docker-compose up --build`), and (b) execution and validation of system integration test cases (TC-01 through TC-05) covering inference, performance latency, and defensive error handling.

---

## 2. Setup & Deployment Verification

| Deployment / Setup Step | Result |
| --- | --- |
| 1. Build and run containers via Docker Compose (`docker compose up --build`) | Worked as written; multi-stage layer caching completed successfully |
| 2. Backend container (`fruit_grading_backend`) initialization | Worked as written; Uvicorn running on `http://0.0.0.0:8000` with ONNX model and labels loaded |
| 3. Presentation tier initialization (Streamlit GUI accessible) | Worked as written; container services linked successfully within the Docker network |

**Conclusion:** The containerized deployment tutorial and Docker configuration worked seamlessly, removing host-environment discrepancies.

---

## 3. System Integration & Functional Test Results

Test Suite: System integration test cases (TC-01 to TC-05) executed against the full-stack containerized environment.

| Test ID | Scenario Description | Test Inputs | Expected Output | Actual Output | Result |
| --- | --- | --- | --- | --- | --- |
| TC-01 | Fresh Apple Classification | Clean image `freshapples_01.jpg` | Label 'Fresh Apple', confidence >85%, Green badge | FRESH, `freshapples`, confidence >90%, Green badge | Pass |
| TC-02 | Rotten Banana Classification | Decaying image `rottenbananas_12.png` | Label 'Rotten Banana', confidence ≥ 90%, Red alert badge | ROTTEN, `rottenbanana`, confidence 100.00%, Red alert badge | Pass |
| TC-03 | REST API Batch Inference | HTTP POST `/predict` with image binary | JSON response `{"class": "freshoranges", "confidence": 0.94}` | Valid HTTP 200 OK with matching JSON fields | Pass |
| TC-04 | Inference Latency Benchmark | 50 sequential prediction requests | Mean execution time per image <200 ms | Mean end-to-end latency of 72.4 ms per image on CPU node | Pass |
| TC-05 | Invalid File Format Handling | Non-image text file `document.txt` | System error: "Invalid image format. Upload .jpg or .png" | Server returns HTTP 400 without crashing | Pass |

**Summary:** 5 of 5 integration test cases passed successfully, validating end-to-end functionality, latency targets, and robust error handling.

---

## 4. Observations

1. The refactored global loading of the ONNX Runtime inference session successfully eliminated connection overhead and memory leaks, ensuring stable concurrent traffic handling during multi-user testing.
2. The integration of Pillow's `ImageOps.fit` preprocessing contract effectively resolved geometric distortion issues previously observed when processing elongated produce like bananas, preserving natural aspect ratios prior to tensor conversion.
3. Performance evaluation across the 15% holdout test split (2,655 images) demonstrated exceptional classification performance, achieving an overall accuracy of 99.96% and a macro F1-score of 0.9997.
4. Operational benchmarks confirmed a mean end-to-end inference latency of 72.4 ms per image (well below the < 200 ms threshold), with ONNX Runtime execution taking 28.5 ms.
5. Defensive error handling mechanisms successfully caught invalid file formats (Alt-1) and prevented application crashes, returning clear validation messages to the user interface.
6. The containerized base image (`python:3.10-slim`) maintained a lightweight footprint of 248 MB while supporting multi-stage layer caching for rapid builds.
7. No unhandled exceptions, server drops, or memory allocation faults occurred during benchmark stress testing, proving system stability under full-stack container orchestration.

---

## 5. Link to Project Goals & Next Steps

The Sprint 2 objectives focused on full-stack integration, ONNX runtime optimization, and single-command Docker deployment. The verification results above confirm that all structural and performance targets have been thoroughly met.

### Additional QA Recommendations

* Implement automated regression scripts within the Docker pipeline to execute test cases TC-01 through TC-05 automatically upon container initialization.
* Expand monitoring logs in `audit_logs.json` to capture detailed client IP metadata and payload sizes for enhanced traceability.
* Maintain the current validation image test bank as a permanent regression suite for future container updates or model weight revisions.

---

## 6. Conclusion

The automated fruit quality grading system has been successfully integrated across all three tiers—Streamlit presentation, FastAPI logic, and ONNX Runtime data access—and containerized via Docker. System testing confirmed outstanding classification metrics (99.96% accuracy) and high-speed execution (72.4 ms latency), satisfying all architectural requirements without compromising system stability.
