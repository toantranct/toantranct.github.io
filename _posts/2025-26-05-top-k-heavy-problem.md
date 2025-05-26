---
title: "System Design Interview: Top K Heavy Hitters - Từ Cơ Bản Đến Tối Ưu"
date: 2025-05-23 09:00:00 +0700
categories: [System Design, Interview]
tags: [system-design, top-k, heavy-hitters, scalability, distributed-systems, interview]
---


## Đặt Vấn Đề

**Top K Heavy Hitters** là một trong những bài toán kinh điển trong **System Design Interview**. Bài toán yêu cầu xác định **top K trending events** trong các khoảng thời gian cụ thể như:

- **Hashtags** phổ biến nhất trên LinkedIn
- **Photos** được like nhiều nhất trên Instagram  
- **Songs** được phát nhiều nhất trên Spotify

Trong các time frame: **1 phút**, **5 phút**, **1 giờ**, và **1 ngày**.

## Yêu Cầu Hệ Thống

### Functional Requirements
- Tìm **top K heavy hitters/events** trong các khoảng thời gian: 1 phút, 5 phút, 1 giờ, 1 ngày

### Non-Functional Requirements

**🔄 Highly Scalable**
- Xử lý được lượng events khổng lồ: **hàng tỷ events/ngày**
- Có khả năng mở rộng theo chiều ngang

**⚡ High Performance**  
- **Low latency**: Trả về top K list trong vài milliseconds
- **Real-time processing** cho các kết quả nhanh

**🛡️ Highly Available**
- Đảm bảo uptime cao, không gián đoạn service
- **Fault tolerance** khi có sự cố

## Phương Pháp Tiếp Cận

Chúng ta sẽ bắt đầu từ solution đơn giản nhất, sau đó phân tích **pros/cons** và dần tối ưu để đáp ứng các yêu cầu về **scalability**, **availability**, và **performance**.

## Solution 1: Hashmap và Heap (Single Server)

### Ý Tưởng
- Sử dụng **HashMap** để lưu frequency của events
- Sử dụng **Min Heap** kích thước K để maintain top K events
- **In-memory processing** cho tốc độ cao

### Architecture Diagram

<svg width="800" height="400" viewBox="0 0 800 400" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="400" fill="#f8f9fa"/>
  
  <!-- Event Stream -->
  <rect x="50" y="80" width="120" height="60" rx="10" fill="#e3f2fd" stroke="#1976d2" stroke-width="2"/>
  <text x="110" y="105" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Event Stream</text>
  <text x="110" y="120" text-anchor="middle" font-family="Arial" font-size="10">Events A, B, C...</text>
  
  <!-- Arrow 1 -->
  <path d="M 170 110 L 230 110" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Single Server -->
  <rect x="250" y="50" width="300" height="200" rx="15" fill="#fff3e0" stroke="#f57c00" stroke-width="3"/>
  <text x="400" y="35" text-anchor="middle" font-family="Arial" font-size="14" font-weight="bold">Single Server</text>
  
  <!-- HashMap -->
  <rect x="270" y="80" width="120" height="80" rx="8" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="330" y="100" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">HashMap</text>
  <text x="330" y="115" text-anchor="middle" font-family="Arial" font-size="10">Event → Count</text>
  <text x="330" y="130" text-anchor="middle" font-family="Arial" font-size="9">A: 5, B: 3, C: 8</text>
  <text x="330" y="145" text-anchor="middle" font-family="Arial" font-size="9">In-Memory</text>
  
  <!-- Min Heap -->
  <rect x="410" y="80" width="120" height="80" rx="8" fill="#fce4ec" stroke="#e91e63" stroke-width="2"/>
  <text x="470" y="100" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Min Heap</text>
  <text x="470" y="115" text-anchor="middle" font-family="Arial" font-size="10">Size K</text>
  <text x="470" y="130" text-anchor="middle" font-family="Arial" font-size="9">Top K Events</text>
  <text x="470" y="145" text-anchor="middle" font-family="Arial" font-size="9">[C:8, A:5, B:3]</text>
  
  <!-- Processing Logic -->
  <rect x="270" y="180" width="260" height="50" rx="8" fill="#f3e5f5" stroke="#9c27b0" stroke-width="2"/>
  <text x="400" y="200" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Processing Logic</text>
  <text x="400" y="215" text-anchor="middle" font-family="Arial" font-size="9">Update HashMap → Check Top K → Update Heap</text>
  
  <!-- Arrow 2 -->
  <path d="M 550 140 L 610 140" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Output -->
  <rect x="630" y="110" width="120" height="60" rx="10" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="690" y="130" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Top K Results</text>
  <text x="690" y="145" text-anchor="middle" font-family="Arial" font-size="10">Low Latency</text>
  <text x="690" y="160" text-anchor="middle" font-family="Arial" font-size="10">Real-time</text>
  
  <!-- Performance Indicators -->
  <rect x="50" y="280" width="700" height="80" rx="10" fill="#f1f8e9" stroke="#8bc34a" stroke-width="2"/>
  <text x="400" y="300" text-anchor="middle" font-family="Arial" font-size="14" font-weight="bold">Performance Characteristics</text>
  <text x="200" y="320" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold" fill="#4caf50">✓ Pros</text>
  <text x="200" y="335" text-anchor="middle" font-family="Arial" font-size="10">• Low latency (~ms)</text>
  <text x="200" y="350" text-anchor="middle" font-family="Arial" font-size="10">• Simple implementation</text>
  
  <text x="600" y="320" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold" fill="#f44336">✗ Cons</text>
  <text x="600" y="335" text-anchor="middle" font-family="Arial" font-size="10">• Not scalable</text>
  <text x="600" y="350" text-anchor="middle" font-family="Arial" font-size="10">• Memory limitation</text>
  
  <!-- Arrow marker definition -->
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#333"/>
    </marker>
  </defs>
</svg>

### Cách Hoạt Động
```python
# Pseudo code
hashmap = {}  # event -> frequency
min_heap = MinHeap(size=K)

def process_event(event):
    # Update frequency
    hashmap[event] = hashmap.get(event, 0) + 1
    
    # Update top K if needed
    if hashmap[event] impacts top K:
        min_heap.update(event, hashmap[event])

def get_top_k():
    return min_heap.get_all()
```

### Đánh Giá
✅ **Pros**: 
- **Low latency** nhờ in-memory processing
- Đơn giản, dễ implement

❌ **Cons**:
- **Không scalable**: Không thể handle hàng tỷ events trên single server
- **Memory limitation**: HashMap sẽ quá lớn

![Hashmap & Heap Single Server](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*09nKkfgaRkj7nlAWXHFiuw.png)

## Solution 2: Horizontal Scaling với Multiple Servers

### Ý Tưởng
- **Partition events** across multiple servers bằng **shard key**
- Mỗi server maintain HashMap và Min Heap riêng
- **Merge results** từ tất cả servers để có final top K

### Architecture Diagram

<svg width="900" height="500" viewBox="0 0 900 500" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="900" height="500" fill="#f8f9fa"/>
  
  <!-- Event Stream -->
  <rect x="50" y="100" width="120" height="60" rx="10" fill="#e3f2fd" stroke="#1976d2" stroke-width="2"/>
  <text x="110" y="125" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Event Stream</text>
  <text x="110" y="140" text-anchor="middle" font-family="Arial" font-size="10">Billions/day</text>
  
  <!-- Load Balancer -->
  <rect x="200" y="100" width="100" height="60" rx="10" fill="#fff3e0" stroke="#ff9800" stroke-width="2"/>
  <text x="250" y="120" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Load</text>
  <text x="250" y="135" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Balancer</text>
  <text x="250" y="150" text-anchor="middle" font-family="Arial" font-size="9">Shard Key</text>
  
  <!-- Arrows from Event to Load Balancer -->
  <path d="M 170 130 L 200 130" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Server 1 -->
  <rect x="350" y="50" width="150" height="120" rx="10" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="425" y="40" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Server 1</text>
  <rect x="365" y="70" width="60" height="40" rx="5" fill="#c8e6c9" stroke="#4caf50"/>
  <text x="395" y="85" text-anchor="middle" font-family="Arial" font-size="9">HashMap</text>
  <text x="395" y="100" text-anchor="middle" font-family="Arial" font-size="8">Events A-F</text>
  <rect x="440" y="70" width="60" height="40" rx="5" fill="#ffcdd2" stroke="#f44336"/>
  <text x="470" y="85" text-anchor="middle" font-family="Arial" font-size="9">MinHeap</text>
  <text x="470" y="100" text-anchor="middle" font-family="Arial" font-size="8">Top K₁</text>
  
  <!-- Server 2 -->
  <rect x="350" y="190" width="150" height="120" rx="10" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="425" y="180" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Server 2</text>
  <rect x="365" y="210" width="60" height="40" rx="5" fill="#c8e6c9" stroke="#4caf50"/>
  <text x="395" y="225" text-anchor="middle" font-family="Arial" font-size="9">HashMap</text>
  <text x="395" y="240" text-anchor="middle" font-family="Arial" font-size="8">Events G-M</text>
  <rect x="440" y="210" width="60" height="40" rx="5" fill="#ffcdd2" stroke="#f44336"/>
  <text x="470" y="225" text-anchor="middle" font-family="Arial" font-size="9">MinHeap</text>
  <text x="470" y="240" text-anchor="middle" font-family="Arial" font-size="8">Top K₂</text>
  
  <!-- Server 3 -->
  <rect x="350" y="330" width="150" height="120" rx="10" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="425" y="320" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Server 3</text>
  <rect x="365" y="350" width="60" height="40" rx="5" fill="#c8e6c9" stroke="#4caf50"/>
  <text x="395" y="365" text-anchor="middle" font-family="Arial" font-size="9">HashMap</text>
  <text x="395" y="380" text-anchor="middle" font-family="Arial" font-size="8">Events N-Z</text>
  <rect x="440" y="350" width="60" height="40" rx="5" fill="#ffcdd2" stroke="#f44336"/>
  <text x="470" y="365" text-anchor="middle" font-family="Arial" font-size="9">MinHeap</text>
  <text x="470" y="380" text-anchor="middle" font-family="Arial" font-size="8">Top K₃</text>
  
  <!-- Arrows from Load Balancer to Servers -->
  <path d="M 300 115 L 350 90" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 300 130 L 350 230" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 300 145 L 350 370" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Merging Service -->
  <rect x="550" y="200" width="120" height="80" rx="10" fill="#f3e5f5" stroke="#9c27b0" stroke-width="2"/>
  <text x="610" y="220" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Merging</text>
  <text x="610" y="235" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Service</text>
  <text x="610" y="255" text-anchor="middle" font-family="Arial" font-size="9">Combine K₁,K₂,K₃</text>
  <text x="610" y="270" text-anchor="middle" font-family="Arial" font-size="9">→ Final Top K</text>
  
  <!-- Arrows from Servers to Merging Service -->
  <path d="M 500 110 L 550 220" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 500 250 L 550 240" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 500 390 L 550 260" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Database -->
  <rect x="720" y="210" width="120" height="60" rx="10" fill="#e1f5fe" stroke="#03a9f4" stroke-width="2"/>
  <text x="780" y="235" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Database</text>
  <text x="780" y="250" text-anchor="middle" font-family="Arial" font-size="10">Final Top K</text>
  <text x="780" y="265" text-anchor="middle" font-family="Arial" font-size="10">Results</text>
  
  <!-- Arrow from Merging Service to Database -->
  <path d="M 670 240 L 720 240" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Time Window Indicator -->
  <rect x="50" y="400" width="200" height="50" rx="8" fill="#fffde7" stroke="#fbc02d" stroke-width="2"/>
  <text x="150" y="420" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Time Window End</text>
  <text x="150" y="435" text-anchor="middle" font-family="Arial" font-size="9">Reset & Start Next Window</text>
  
  <!-- Problems Box -->
  <rect x="550" y="350" width="290" height="100" rx="10" fill="#ffebee" stroke="#f44336" stroke-width="2"/>
  <text x="695" y="370" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold" fill="#d32f2f">Remaining Problems</text>
  <text x="570" y="390" font-family="Arial" font-size="10">• Still not fully scalable</text>
  <text x="570" y="405" font-family="Arial" font-size="10">• Expensive horizontal scaling</text>
  <text x="570" y="420" font-family="Arial" font-size="10">• Complex result merging</text>
  <text x="570" y="435" font-family="Arial" font-size="10">• What if events keep increasing?</text>
  
  <!-- Arrow marker definition -->
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#333"/>
    </marker>
  </defs>
</svg>

### Cách Hoạt Động
```
Events → Load Balancer → Server 1 (HashMap + MinHeap)
                     → Server 2 (HashMap + MinHeap)  
                     → Server 3 (HashMap + MinHeap)
                     ↓
                   Merging Service → Final Top K → Database
```

### Cách Hoạt Động
1. Events được **hash** và phân phối đến các servers
2. Mỗi server xử lý subset của events
3. Cuối mỗi time window, servers trả về top K lists của họ
4. **Merging service** combine các lists thành final result

### Đánh Giá
✅ **Pros**: 
- Cải thiện throughput
- Phân tán load

❌ **Cons**:
- Vẫn **không hoàn toàn scalable** khi số events tăng liên tục
- **Expensive** khi cần thêm nhiều servers
- **Complexity** trong việc merge results

![Horizontally scaling initial solution](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*RszcROBKhk6Hg6BwUdeeeA.png)

## Solution 3: Giải Pháp Hai Phần

Do horizontal scaling vẫn có limitations, chúng ta chia thành **2 parts** để giải quyết các vấn đề khác nhau:

### Part 1: Count Min Sketch + Min Heap (Real-time, Approximate)

#### Count Min Sketch là gì?
**Count Min Sketch** là một **probabilistic data structure** dùng để estimate frequency của events trong data stream:

- **Fixed size**: Không phụ thuộc vào số lượng events
- **Space efficient**: Tiết kiệm memory hơn HashMap
- **Approximate results**: Có thể overcount do hash collisions

#### Cách Hoạt Động
```python
# Count Min Sketch matrix: rows × cols
# rows = số hash functions, cols = buckets per hash function
sketch = [[0] * cols for _ in range(rows)]

def update_event(event):
    frequencies = []
    for i in range(rows):
        hash_val = hash_functions[i](event) % cols
        sketch[i][hash_val] += 1
        frequencies.append(sketch[i][hash_val])
    
    # Lấy minimum frequency (tránh overcount)
    estimated_freq = min(frequencies)
    
    # Update min heap nếu cần
    if estimated_freq impacts top K:
        min_heap.update(event, estimated_freq)
```

#### Ví Dụ Minh Họa
Giả sử có **Count Min Sketch** kích thước 3×7 (3 hash functions):

**Event A xuất hiện:**
- Hash1(A) % 7 = 2 → sketch[0][2]++
- Hash2(A) % 7 = 5 → sketch[1][5]++  
- Hash3(A) % 7 = 1 → sketch[2][1]++
- Frequency(A) = min(sketch[0][2], sketch[1][5], sketch[2][1])

#### Đánh Giá Part 1
✅ **Pros**:
- **Real-time** và fast
- **Fixed memory** footprint
- **Cost effective**
- **Simple** implementation

❌ **Cons**:
- **Approximate results** (không chính xác 100%)

![Count Min Sketch & Min Heap Single Server Solution](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*m3jiQLExKhdMt_nmeUzq2A.png)

### Part 2: HDFS + MapReduce (Batch, Exact)

#### Ý Tưởng  
- Lưu **event logs** trong **distributed file system** (HDFS)
- Sử dụng **MapReduce jobs** để xử lý batch

#### HDFS + MapReduce Architecture

<svg width="900" height="500" viewBox="0 0 900 500" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="900" height="500" fill="#f8f9fa"/>
  
  <!-- Title -->
  <text x="450" y="25" text-anchor="middle" font-family="Arial" font-size="16" font-weight="bold">HDFS + MapReduce Solution (Batch Processing)</text>
  
  <!-- Event Stream -->
  <rect x="50" y="60" width="100" height="50" rx="8" fill="#e3f2fd" stroke="#1976d2" stroke-width="2"/>
  <text x="100" y="80" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Event</text>
  <text x="100" y="95" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Stream</text>
  
  <!-- Kafka Queue -->
  <rect x="200" y="60" width="100" height="50" rx="8" fill="#fff3e0" stroke="#ff9800" stroke-width="2"/>
  <text x="250" y="80" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Kafka</text>
  <text x="250" y="95" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Queue</text>
  
  <!-- HDFS Consumer -->
  <rect x="350" y="60" width="100" height="50" rx="8" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="400" y="80" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">HDFS</text>
  <text x="400" y="95" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Consumer</text>
  
  <!-- HDFS Storage -->
  <rect x="500" y="40" width="120" height="90" rx="10" fill="#f3e5f5" stroke="#9c27b0" stroke-width="2"/>
  <text x="560" y="60" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">HDFS Storage</text>
  <rect x="515" y="70" width="90" height="15" rx="3" fill="#ce93d8"/>
  <text x="560" y="81" text-anchor="middle" font-family="Arial" font-size="8">event_logs_01.txt</text>
  <rect x="515" y="90" width="90" height="15" rx="3" fill="#ce93d8"/>
  <text x="560" y="101" text-anchor="middle" font-family="Arial" font-size="8">event_logs_02.txt</text>
  <rect x="515" y="110" width="90" height="15" rx="3" fill="#ce93d8"/>
  <text x="560" y="121" text-anchor="middle" font-family="Arial" font-size="8">event_logs_03.txt</text>
  
  <!-- Arrows -->
  <path d="M 150 85 L 200 85" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 300 85 L 350 85" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 450 85 L 500 85" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- MapReduce Section -->
  <rect x="50" y="180" width="800" height="280" rx="15" fill="#f1f8e9" stroke="#8bc34a" stroke-width="3"/>
  <text x="450" y="205" text-anchor="middle" font-family="Arial" font-size="14" font-weight="bold">MapReduce Processing Pipeline</text>
  
  <!-- MapReduce Job 1 -->
  <rect x="80" y="230" width="350" height="100" rx="10" fill="#fff3e0" stroke="#ff9800" stroke-width="2"/>
  <text x="255" y="250" text-anchor="middle" font-family="Arial" font-size="13" font-weight="bold">MapReduce Job 1: Frequency Counting</text>
  
  <!-- Map Phase 1 -->
  <rect x="100" y="270" width="140" height="50" rx="5" fill="#ffcc80" stroke="#f57c00"/>
  <text x="170" y="285" text-anchor="middle" font-family="Arial" font-size="10" font-weight="bold">Map Phase</text>
  <text x="110" y="300" font-family="Arial" font-size="8">Input: event_logs</text>
  <text x="110" y="310" font-family="Arial" font-size="8">Output: (event, 1)</text>
  
  <!-- Reduce Phase 1 -->
  <rect x="270" y="270" width="140" height="50" rx="5" fill="#ff8a65" stroke="#d84315"/>
  <text x="340" y="285" text-anchor="middle" font-family="Arial" font-size="10" font-weight="bold">Reduce Phase</text>
  <text x="280" y="300" font-family="Arial" font-size="8">Input: (event, [1,1,1...])</text>
  <text x="280" y="310" font-family="Arial" font-size="8">Output: (event, count)</text>
  
  <!-- Arrow between Map and Reduce 1 -->
  <path d="M 240 295 L 270 295" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- MapReduce Job 2 -->
  <rect x="470" y="230" width="350" height="100" rx="10" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="645" y="250" text-anchor="middle" font-family="Arial" font-size="13" font-weight="bold">MapReduce Job 2: Top K Selection</text>
  
  <!-- Map Phase 2 -->
  <rect x="490" y="270" width="140" height="50" rx="5" fill="#a5d6a7" stroke="#388e3c"/>
  <text x="560" y="285" text-anchor="middle" font-family="Arial" font-size="10" font-weight="bold">Map Phase</text>
  <text x="500" y="300" font-family="Arial" font-size="8">Input: (event, count)</text>
  <text x="500" y="310" font-family="Arial" font-size="8">Output: ("all", (event, count))</text>
  
  <!-- Reduce Phase 2 -->
  <rect x="660" y="270" width="140" height="50" rx="5" fill="#66bb6a" stroke="#2e7d32"/>
  <text x="730" y="285" text-anchor="middle" font-family="Arial" font-size="10" font-weight="bold">Reduce Phase</text>
  <text x="670" y="300" font-family="Arial" font-size="8">Input: All (event, count)</text>
  <text x="670" y="310" font-family="Arial" font-size="8">Output: Top K events</text>
  
  <!-- Arrow between Map and Reduce 2 -->
  <path d="M 630 295 L 660 295" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Arrow between Job 1 and Job 2 -->
  <path d="M 430 280 L 470 280" stroke="#333" stroke-width="3" marker-end="url(#arrowhead)" stroke="#d32f2f"/>
  <text x="450" y="270" text-anchor="middle" font-family="Arial" font-size="9" font-weight="bold" fill="#d32f2f">Intermediate</text>
  <text x="450" y="285" text-anchor="middle" font-family="Arial" font-size="9" font-weight="bold" fill="#d32f2f">Results</text>
  
  <!-- Example Data Flow -->
  <rect x="80" y="350" width="740" height="80" rx="8" fill="#e1f5fe" stroke="#0288d1" stroke-width="2"/>
  <text x="450" y="370" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Example Data Flow</text>
  
  <text x="100" y="390" font-family="Arial" font-size="10"><tspan font-weight="bold">Input:</tspan> event_logs → [A, B, A, C, A, B, C, C, C]</text>
  <text x="100" y="405" font-family="Arial" font-size="10"><tspan font-weight="bold">Job 1 Output:</tspan> A:3, B:2, C:4</text>
  <text x="100" y="420" font-family="Arial" font-size="10"><tspan font-weight="bold">Job 2 Output (K=3):</tspan> [C:4, A:3, B:2]</text>
  
  <!-- Database -->
  <rect x="700" y="60" width="120" height="60" rx="10" fill="#e1f5fe" stroke="#03a9f4" stroke-width="2"/>
  <text x="760" y="80" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Database</text>
  <text x="760" y="95" text-anchor="middle" font-family="Arial" font-size="10">Exact Results</text>
  <text x="760" y="110" text-anchor="middle" font-family="Arial" font-size="10">Batch Updates</text>
  
  <!-- Arrow from Job 2 to Database -->
  <path d="M 730 320 Q 760 350 760 120" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)" fill="none"/>
  
  <!-- Timing Information -->
  <rect x="650" y="380" width="180" height="50" rx="8" fill="#fff9c4" stroke="#f57f17" stroke-width="2"/>
  <text x="740" y="395" text-anchor="middle" font-family="Arial" font-size="10" font-weight="bold">Execution Schedule</text>
  <text x="740" y="408" text-anchor="middle" font-family="Arial" font-size="9">Hourly/Daily Batch Jobs</text>
  <text x="740" y="420" text-anchor="middle" font-family="Arial" font-size="9">High Latency, High Accuracy</text>
  
  <!-- Arrow marker definition -->
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#333"/>
    </marker>
  </defs>
</svg>

#### Architecture Flow
```
Events → Kafka Queue → HDFS Consumer → HDFS Storage
                                    ↓
                              MapReduce Job 1: Count frequencies
                                    ↓  
                              MapReduce Job 2: Find top K
                                    ↓
                                Database
```

#### MapReduce Jobs
**Job 1 - Frequency Counting:**
```python
# Map phase
def map(event_log):
    for event in parse(event_log):
        if event.timestamp in target_window:
            emit(event.id, 1)

# Reduce phase  
def reduce(event_id, counts):
    emit(event_id, sum(counts))
```

**Job 2 - Top K Selection:**
```python
# Map phase
def map(event_id, frequency):
    emit("all", (event_id, frequency))

# Reduce phase
def reduce(key, event_frequencies):
    top_k = heapq.nlargest(K, event_frequencies, key=lambda x: x[1])
    emit("top_k_result", top_k)
```

#### Đánh Giá Part 2
✅ **Pros**:
- **Exact results** (chính xác 100%)
- **Highly scalable** với distributed processing
- **Cost effective** cho batch processing

❌ **Cons**:
- **Not real-time** (độ trễ cao)
- **Complex setup** và maintenance

![HDFS and Map Reduce Job Solution](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*VA9r2107SLBeQcwPgv-hww.png)

## Solution 4: Final Combined Solution

### Final Combined Architecture

<svg width="1000" height="600" viewBox="0 0 1000 600" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="1000" height="600" fill="#f8f9fa"/>
  
  <!-- Title -->
  <text x="500" y="25" text-anchor="middle" font-family="Arial" font-size="16" font-weight="bold">Final Combined Solution: Real-time + Batch Processing</text>
  
  <!-- Event Stream -->
  <rect x="50" y="80" width="120" height="60" rx="10" fill="#e3f2fd" stroke="#1976d2" stroke-width="2"/>
  <text x="110" y="105" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Event Stream</text>
  <text x="110" y="120" text-anchor="middle" font-family="Arial" font-size="10">Billions/day</text>
  
  <!-- API Gateway -->
  <rect x="220" y="80" width="120" height="60" rx="10" fill="#fff3e0" stroke="#ff9800" stroke-width="2"/>
  <text x="280" y="105" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">API Gateway</text>
  <text x="280" y="120" text-anchor="middle" font-family="Arial" font-size="10">Event Routing</text>
  
  <!-- Kafka -->
  <rect x="390" y="80" width="120" height="60" rx="10" fill="#f3e5f5" stroke="#9c27b0" stroke-width="2"/>
  <text x="450" y="105" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Kafka Queue</text>
  <text x="450" y="120" text-anchor="middle" font-family="Arial" font-size="10">Message Broker</text>
  
  <!-- Arrows to Kafka -->
  <path d="M 170 110 L 220 110" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 340 110 L 390 110" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Path 1: Real-time Processing -->
  <rect x="100" y="200" width="400" height="180" rx="15" fill="#e8f5e8" stroke="#4caf50" stroke-width="3"/>
  <text x="300" y="225" text-anchor="middle" font-family="Arial" font-size="14" font-weight="bold">Path 1: Real-time Processing</text>
  
  <!-- Count Min Sketch Server -->
  <rect x="130" y="250" width="150" height="100" rx="10" fill="#c8e6c9" stroke="#388e3c" stroke-width="2"/>
  <text x="205" y="270" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Count Min Sketch</text>
  <text x="205" y="285" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Processor</text>
  
  <!-- CMS Matrix -->
  <rect x="140" y="295" width="50" height="30" rx="3" fill="#a5d6a7"/>
  <text x="165" y="310" text-anchor="middle" font-family="Arial" font-size="8">Matrix</text>
  <text x="165" y="320" text-anchor="middle" font-family="Arial" font-size="8">3×7</text>
  
  <!-- Min Heap -->
  <rect x="200" y="295" width="50" height="30" rx="3" fill="#ffcdd2"/>
  <text x="225" y="310" text-anchor="middle" font-family="Arial" font-size="8">MinHeap</text>
  <text x="225" y="320" text-anchor="middle" font-family="Arial" font-size="8">Top K</text>
  
  <!-- Real-time Database -->
  <rect x="320" y="270" width="150" height="60" rx="10" fill="#e1f5fe" stroke="#0288d1" stroke-width="2"/>
  <text x="395" y="290" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Database</text>
  <text x="395" y="305" text-anchor="middle" font-family="Arial" font-size="10">(Real-time Results)</text>
  <text x="395" y="320" text-anchor="middle" font-family="Arial" font-size="9">~95-99% Accuracy</text>
  
  <!-- Arrow from CMS to DB -->
  <path d="M 280 300 L 320 300" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Path 2: Batch Processing -->
  <rect x="550" y="200" width="400" height="180" rx="15" fill="#fff3e0" stroke="#f57c00" stroke-width="3"/>
  <text x="750" y="225" text-anchor="middle" font-family="Arial" font-size="14" font-weight="bold">Path 2: Batch Processing</text>
  
  <!-- HDFS -->
  <rect x="580" y="250" width="100" height="60" rx="10" fill="#f3e5f5" stroke="#9c27b0" stroke-width="2"/>
  <text x="630" y="270" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">HDFS</text>
  <text x="630" y="285" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Storage</text>
  <rect x="590" y="295" width="80" height="8" rx="2" fill="#ce93d8"/>
  <rect x="590" y="305" width="80" height="8" rx="2" fill="#ce93d8"/>
  
  <!-- MapReduce -->
  <rect x="700" y="250" width="100" height="60" rx="10" fill="#ffcc80" stroke="#f57c00" stroke-width="2"/>
  <text x="750" y="270" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">MapReduce</text>
  <text x="750" y="285" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Jobs</text>
  <text x="750" y="300" text-anchor="middle" font-family="Arial" font-size="9">1. Count Freq</text>
  <text x="750" y="310" text-anchor="middle" font-family="Arial" font-size="9">2. Top K</text>
  
  <!-- Batch Database -->
  <rect x="820" y="250" width="100" height="60" rx="10" fill="#e1f5fe" stroke="#0288d1" stroke-width="2"/>
  <text x="870" y="270" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Database</text>
  <text x="870" y="285" text-anchor="middle" font-family="Arial" font-size="10">(Exact Results)</text>
  <text x="870" y="300" text-anchor="middle" font-family="Arial" font-size="9">100% Accuracy</text>
  
  <!-- Arrows in Path 2 -->
  <path d="M 680 280 L 700 280" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 800 280 L 820 280" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Arrows from Kafka to Paths -->
  <path d="M 450 140 Q 300 170 300 200" stroke="#4caf50" stroke-width="3" marker-end="url(#arrowhead)" fill="none"/>
  <path d="M 450 140 Q 630 170 630 200" stroke="#f57c00" stroke-width="3" marker-end="url(#arrowhead)" fill="none"/>
  
  <!-- Path Labels -->
  <text x="350" y="165" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold" fill="#4caf50">Real-time</text>
  <text x="580" y="165" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold" fill="#f57c00">Batch</text>
  
  <!-- Serving Layer -->
  <rect x="200" y="420" width="600" height="120" rx="15" fill="#f1f8e9" stroke="#8bc34a" stroke-width="3"/>
  <text x="500" y="445" text-anchor="middle" font-family="Arial" font-size="14" font-weight="bold">Unified Serving Layer</text>
  
  <!-- Query Interface -->
  <rect x="230" y="470" width="160" height="50" rx="8" fill="#fff9c4" stroke="#f57f17" stroke-width="2"/>
  <text x="310" y="485" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Query Interface</text>
  <text x="310" y="500" text-anchor="middle" font-family="Arial" font-size="9">get_top_k(window, mode)</text>
  <text x="310" y="515" text-anchor="middle" font-family="Arial" font-size="9">mode: fast | accurate | hybrid</text>
  
  <!-- Result Merger -->
  <rect x="420" y="470" width="160" height="50" rx="8" fill="#e8eaf6" stroke="#5c6bc0" stroke-width="2"/>
  <text x="500" y="485" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Result Merger</text>
  <text x="500" y="500" text-anchor="middle" font-family="Arial" font-size="9">Combine & Validate</text>
  <text x="500" y="515" text-anchor="middle" font-family="Arial" font-size="9">Confidence Scoring</text>
  
  <!-- Response Cache -->
  <rect x="610" y="470" width="160" height="50" rx="8" fill="#fce4ec" stroke="#ec407a" stroke-width="2"/>
  <text x="690" y="485" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Response Cache</text>
  <text x="690" y="500" text-anchor="middle" font-family="Arial" font-size="9">Redis/Memcached</text>
  <text x="690" y="515" text-anchor="middle" font-family="Arial" font-size="9">TTL-based</text>
  
  <!-- Arrows in Serving Layer -->
  <path d="M 390 495 L 420 495" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  <path d="M 580 495 L 610 495" stroke="#333" stroke-width="2" marker-end="url(#arrowhead)"/>
  
  <!-- Connections from Databases to Serving Layer -->
  <path d="M 395 330 Q 395 375 310 420" stroke="#0288d1" stroke-width="2" marker-end="url(#arrowhead)" fill="none"/>
  <path d="M 870 310 Q 870 375 690 420" stroke="#0288d1" stroke-width="2" marker-end="url(#arrowhead)" fill="none"/>
  
  <!-- Performance Metrics -->
  <rect x="50" y="420" width="120" height="120" rx="10" fill="#f3e5f5" stroke="#9c27b0" stroke-width="2"/>
  <text x="110" y="440" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Metrics</text>
  <text x="70" y="460" font-family="Arial" font-size="9">• Latency: ~ms</text>
  <text x="70" y="475" font-family="Arial" font-size="9">• Throughput: 10K+/s</text>
  <text x="70" y="490" font-family="Arial" font-size="9">• Accuracy: 95-100%</text>
  <text x="70" y="505" font-family="Arial" font-size="9">• Availability: 99.9%</text>
  <text x="70" y="520" font-family="Arial" font-size="9">• Scalability: ✓</text>
  
  <!-- Optimizations Box -->
  <rect x="830" y="420" width="140" height="120" rx="10" fill="#e8f5e8" stroke="#4caf50" stroke-width="2"/>
  <text x="900" y="440" text-anchor="middle" font-family="Arial" font-size="11" font-weight="bold">Optimizations</text>
  <text x="845" y="460" font-family="Arial" font-size="9">• Event Aggregation</text>
  <text x="845" y="475" font-family="Arial" font-size="9">• Data Partitioning</text>
  <text x="845" y="490" font-family="Arial" font-size="9">• Caching Layers</text>
  <text x="845" y="505" font-family="Arial" font-size="9">• Auto Scaling</text>
  <text x="845" y="520" font-family="Arial" font-size="9">• Load Balancing</text>
  
  <!-- Arrow marker definition -->
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#333"/>
    </marker>
  </defs>
</svg>

### Kiến Trúc Tổng Hợp
Kết hợp **Part 1** và **Part 2** để có được cả **real-time approximate** và **batch exact** results:

```
                            API Gateway
                                 ↓
                               Kafka
                            ↙         ↘
                    Path 1              Path 2
            Count Min Sketch         HDFS Consumer
            (Real-time)                   ↓
                 ↓                      HDFS
              Database              MapReduce Jobs
                                        ↓  
                                    Database
```

### Detailed Flow

#### Common Components
1. **API Gateway**: Route incoming events
2. **Kafka**: Message queue cho scalable event processing  
3. **Database**: Store results (SQL/NoSQL)

#### Processing Paths

**Path 1 - Real-time Processing:**
1. Events từ Kafka → **Count Min Sketch processor**
2. Calculate approximate top K real-time
3. Store results vào database với timestamp

**Path 2 - Batch Processing:**  
1. Events từ Kafka → **HDFS consumer**
2. Store event data trong HDFS
3. **Hourly MapReduce jobs**:
   - Job 1: Aggregate event frequencies
   - Job 2: Calculate exact top K
4. Store precise results vào database

### Serving Layer
```python
def get_top_k(time_window, precision="fast"):
    if precision == "fast":
        return get_approximate_results(time_window)
    elif precision == "accurate":  
        return get_exact_results(time_window)
    else:
        # Combine both for confidence scoring
        approx = get_approximate_results(time_window)
        exact = get_exact_results(time_window)
        return merge_with_confidence(approx, exact)
```

![Final Solution](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*BUdKYpYB8VNaOYLZevCesw.png)

## Tối Ưu Hóa Nâng Cao

### 1. Event Aggregation tại API Gateway
```python
# Pre-aggregate events before sending to Kafka
def aggregate_events(events_batch):
    aggregated = {}
    for event in events_batch:
        key = (event.id, event.time_window)
        aggregated[key] = aggregated.get(key, 0) + 1
    
    return [Event(id=k[0], count=v, window=k[1]) 
            for k, v in aggregated.items()]
```

### 2. Data Partitioner Pattern
```
Kafka → Data Partitioner → Partition by (event_id + time_window)
                    ↓
               Partitioned Kafka Topics
                    ↓  
              Partition Processors
```

**Benefits:**
- **Better load balancing**
- **Ordered processing** per partition
- **Reduced hotspots**

### 3. Caching Layer
```python
# Multi-level caching
L1_Cache = {}  # In-memory for frequent queries
L2_Cache = Redis()  # Distributed cache

def get_cached_top_k(time_window):
    # Try L1 first
    if time_window in L1_Cache:
        return L1_Cache[time_window]
    
    # Try L2
    result = L2_Cache.get(f"top_k_{time_window}")
    if result:
        L1_Cache[time_window] = result  # Populate L1
        return result
    
    # Fetch from database
    result = database.get_top_k(time_window)
    
    # Populate both caches
    L2_Cache.set(f"top_k_{time_window}", result, ttl=300)
    L1_Cache[time_window] = result
    
    return result
```

### 4. Time Window Management
```python
class TimeWindowManager:
    def __init__(self):
        self.windows = {
            "1min": SlidingWindow(60),
            "5min": SlidingWindow(300), 
            "1hour": SlidingWindow(3600),
            "1day": SlidingWindow(86400)
        }
    
    def process_event(self, event):
        current_time = time.time()
        for window_name, window in self.windows.items():
            if window.should_process(event.timestamp, current_time):
                window.add_event(event)
                
            if window.should_emit_results(current_time):
                top_k = window.get_top_k()
                self.emit_results(window_name, top_k)
```

## Monitoring và Metrics

### Key Metrics cần theo dõi:

**📊 Performance Metrics:**
- **Latency**: P50, P95, P99 response times
- **Throughput**: Events processed per second
- **Error Rate**: Failed processing percentage

**💾 Resource Metrics:**
- **Memory Usage**: Count Min Sketch memory footprint
- **CPU Utilization**: Processing overhead
- **Network I/O**: Data transfer rates

**🎯 Business Metrics:**
- **Accuracy**: Approximate vs exact results comparison
- **Freshness**: Time lag cho real-time results
- **Coverage**: Percentage of events successfully processed

### Alerting Strategy
```python
# Example monitoring setup
def setup_monitoring():
    alerts = [
        Alert("high_latency", "p95_latency > 100ms"),
        Alert("low_throughput", "events_per_sec < 1000"),
        Alert("high_error_rate", "error_rate > 1%"),
        Alert("memory_usage", "memory_usage > 80%")
    ]
    
    for alert in alerts:
        monitor.add_alert(alert)
```

## Trade-offs và Considerations

### Real-time vs Accuracy 
 
| Aspect | Real-time (Count Min Sketch) | Batch (MapReduce) |
|--------|------------------------------|-------------------|
| **Latency** | Milliseconds | Minutes to Hours |
| **Accuracy** | ~95-99% | 100% |
| **Resource Cost** | Low | Higher |
| **Complexity** | Simple | Complex |

### Scalability Patterns
- **Vertical Scaling**: Tăng compute power cho single machines
- **Horizontal Scaling**: Thêm machines vào cluster
- **Functional Partitioning**: Chia theo event types
- **Temporal Partitioning**: Chia theo time windows

### Cost Optimization
```python
# Dynamic resource allocation
def optimize_resources(current_load):
    if current_load > high_threshold:
        scale_up_count_min_sketch_servers()
        increase_kafka_partitions()
    elif current_load < low_threshold:
        scale_down_resources()
        consolidate_partitions()
```

## Kết Luận

**Top K Heavy Hitters** system design đòi hỏi careful balance giữa **real-time performance** và **accuracy**. Solution cuối cung cấp:

### 🎯 **Hybrid Approach Benefits:**
- **Fast approximate results** cho real-time use cases
- **Accurate batch results** cho analytics và reporting
- **Scalable architecture** handle billions of events
- **Cost-effective** resource utilization

### 🚀 **Key Takeaways:**
1. **Start simple**, sau đó optimize dần dần
2. **Understand trade-offs** giữa speed, accuracy, và cost  
3. **Use appropriate data structures**: Count Min Sketch cho approximation
4. **Leverage distributed systems**: MapReduce cho exact computation
5. **Monitor everything**: Performance, accuracy, và business metrics

### 📈 **Production Considerations:**
- **Fault tolerance**: Handle server failures gracefully
- **Data consistency**: Ensure reliable event processing  
- **Security**: Protect against malicious events
- **Compliance**: Meet data retention và privacy requirements

System này có thể adapt cho nhiều use cases khác nhau từ **social media trending** đến **e-commerce product recommendations**, **financial fraud detection**, và **IoT sensor data analysis**.

Thành công trong System Design Interview không chỉ là code correct solution, mà còn là demonstrate hiểu biết về **scalability principles**, **trade-offs analysis**, và **real-world constraints**! 💪