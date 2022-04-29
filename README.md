# AIRFLOW

### Mục lục 
[1. Các khái niệm cơ bản trong Airflow](#gioi_thieu_airflow)

 - [1.1 Giới thiệu chung](#gioi_thieu_chung)
 - [1.2 Các khái niệm cơ bản](#cac_khai_niem_co_ban)

[2. Cài đặt Airflow tại local](#CaiDat)

[3. Viết chương trình DAG](#Viet_chuong_trinh_DAG)

<a name="gioi_thieu_airflow"></a>
## 1. Các khái niệm cơ bản trong Airflow 
<a name="gioi_thieu_chung"></a>
### 1.1 Giới thiệu chung 
Airflow là một trong những công cụ workflow automation và scheduling systems phổ biến nhất. Airflow quản lý tất cả các jobs bởi DAG (directed acyclic graphs hay đồ thị có hướng), cho phép quản lý các job dễ hiểu nhất.

Một số lý do để sử dụng Airflow:
* **Open source:** xuất phát từ một dự án internal của Airbnb, Airflow dần được phát triển bởi cộng đồng. Hiện tại Airflow được maintained và managed bởi [Apache](https://airflow.apache.org/)
* **Web Interface:** Airflow hỗ trợ giao diện Flask app để quản lý các workflows, và giúp bạn dễ dàng thay đổi, start và stop. 
* **Python Based:** Mỗi DAG config trong Airflow được viết bởi Python, bao gồm cấu hình schedules, script để chạy, một cách linh động.
<a name="cac_khai_niem_co_ban"></a>
### 1.2 Các khái niệm cơ bản
#### DAG (Directed Acyclic Graphs - đồ thị có hướng)
* Trong Airflow, Directed Acyclic Graphs ([DAGs](https://airflow.apache.org/docs/apache-airflow/stable/concepts/dags.html)) được sử dụng để tạo ra các workflows.
* DAG tập hợp các tasks lại với nhau, tổ chức với các mối quan hệ và sự phụ thuộc để cho biết workflow hoạt động như thế nào.
* Bản thân DAG không quan tâm đến những gì xảy ra bên trong các tasks, nó chỉ đơn thuần là quan tâm đến cách thực thi (thứ tự chạy của các tasks, chạy lại bao nhiêu lần).

![dag](https://user-images.githubusercontent.com/63502091/163365469-827b820f-63aa-4d66-b841-bc0f2e158f31.png)

Ví dụ trong hình trên, DAG định nghĩa ra 6 tasks, mối quan hệ của các tasks cũng như thứ tự phải chạy. 
#### Scheduler
* [Scheduler](https://airflow.apache.org/docs/apache-airflow/stable/concepts/scheduler.html) là hệ thống lập thời gian để quyết định thời điểm thực thi các DAG 
* Giám sát các DAGs và kích hoạt các tasks có yếu tố phụ thuộc. 
#### Operators
* [Operators](https://airflow.apache.org/docs/apache-airflow/stable/concepts/operators.html) là các "workers" để chạy các tasks.
* Mỗi operator chạy các tasks cụ thể được viết bởi python function hoặc shell command.
* Một số operator phổ biết như:
  - **BashOperator:** thực thi môt bash command.
  - **PythonOperator:** gọi một hàm python function tùy ý.
  - **EmailOperator:** gửi một email.
#### Tasks
* [Tasks](https://airflow.apache.org/docs/apache-airflow/stable/concepts/tasks.html) là các hoạt động do người dùng định nghĩa và được thực thi bởi các operators.
* Tasks có thể là các python function hay các script bên ngoài ta có thể gọi tới.

***Note:*** Phân biệt **operators** với **tasks**. Tasks định nghĩa *"what to run?"* trong khí đó operators là *"how to run?"*. Ví dụ python function dùng để thông báo kết quả số bản ghi crawl được là **task** và nó được thực thi bằng cách sử dụng PythonOperator là **operator**.
<a name="CaiDat"></a>
## 2. Cài đặt Airflow tại local 
**B1: Tạo môi trường ảo conda**
```
conda create -n airflow python=3.9
```
**B2: Kích hoạt môi trường ảo** 
```
conda activate airflow
```
**B3: Thiết lập vị trí folder airflow**
 ```
export AIRFLOW_HOME=~/airflow/
```
**B4: Cài đặt thư viện Airflow sử dụng constraints file**
```
pip install "apache-airflow==2.2.5" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.2.5/constraints-3.9.txt"
```
**B5: Khởi tạo database**
```
airflow db init
```
**B6: Tạo airflow user**
```
airflow users create \ 
--username admin \
--password CIST2o20 \
--firstname Manh \
--lastname Luong \
--role Admin \
--email lkmanh@cmc.com.vn
```
**B7: Chạy webserver**
```
airflow webserver --port 8080 -D 
```
**B8: Chạy scheduler**
```
airflow scheduler -D
```
### Chú ý
Trước khi chạy webserver và scheduler cần kiểm tra port 8080 và 8793 bằng command line sau:
```
lsof -i:port
```
Nếu các port trên đang được sử dụng -> kill các process đang chạy 
```
kill -9 pid 
```
Kill hết các process trên port 8080 và 8793:
```
kill $(lsof -t -i:8080)
kill $(lsof -t -i:8793)
```
Xóa airflow examples DAGs
* **B1:** Thay load_examples = False trong file airflow.cfg
```
airflow.cfg > load_examples = False
```
* **B2:** Reset database

```
airflow db reset
```

* **B3:** Kill hết các process trên port 8080 và 8793 
```
kill $(lsof -t -i:8080)
kill $(lsof -t -i:8793)
```

* **B4:** Chạy lại webserver và scheduler 
```
airflow webserver --port 8080 -D 
airflow scheduler -D
```
* Khi chạy chương trình trên server, cần kết nối cổng của local với server để truy cập:
```
ssh -N -L 8080:localhost:8080 server_name@host -p port
```
<a name="Viet_chuong_trinh_DAG"></a>
## 3. Viết chương trình DAG (Data Pipeline)
* **Documentation:** [Appache-tutorial](https://airflow.apache.org/docs/apache-airflow/stable/tutorial.html)
* **Youtube:** [How to Write Your First Airflow DAG](https://www.youtube.com/watch?v=mge56uGRagc&list=PLQ5j-FTc2VhBjU4siviNeYRLG5o6Ty46J&index=2)
