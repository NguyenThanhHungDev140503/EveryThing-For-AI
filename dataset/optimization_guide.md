# Hướng dẫn Chi tiết Tối ưu hóa Dataset JSONL cho Fine-tuning Mô hình Ngôn ngữ

**Tác giả:** Manus AI

## Mục lục

## 1. Giới thiệu

Trong bối cảnh phát triển các mô hình ngôn ngữ lớn (LLMs) ngày càng mạnh mẽ, fine-tuning (tinh chỉnh) đã trở thành một kỹ thuật không thể thiếu để tùy chỉnh các mô hình nền tảng cho các tác vụ hoặc miền cụ thể. Tuy nhiên, hiệu suất của một mô hình được fine-tuned phụ thuộc rất nhiều vào chất lượng của dataset huấn luyện. Một dataset kém chất lượng, chứa dữ liệu trùng lặp, phản hồi quá dài, hoặc thiếu sự đa dạng trong câu hỏi có thể dẫn đến mô hình hoạt động không như mong đợi, thậm chí là "học vẹt" hoặc đưa ra các phản hồi không phù hợp.

Tài liệu này cung cấp một hướng dẫn chi tiết, từng bước về cách tối ưu hóa dataset định dạng JSONL, một định dạng phổ biến cho việc fine-tuning các mô hình ngôn ngữ. Các kỹ thuật được trình bày ở đây đã được áp dụng thành công để cải thiện chất lượng dataset, giúp mô hình học hiệu quả hơn và đưa ra các phản hồi chính xác, tự nhiên hơn. Chúng ta sẽ đi sâu vào ba khía cạnh chính: loại bỏ dữ liệu trùng lặp, xử lý các phản hồi quá dài, và tăng cường sự đa dạng của câu hỏi người dùng.

## 2. Tại sao cần tối ưu hóa Dataset của bạn?

Việc tối ưu hóa dataset trước khi fine-tuning là một bước quan trọng, thường bị bỏ qua nhưng lại có tác động lớn đến hiệu suất cuối cùng của mô hình. Dưới đây là những lý do chính:

### 2.1. Tránh "Học vẹt" và Cải thiện Khả năng Tổng quát hóa

Nếu dataset chứa quá nhiều bản ghi trùng lặp hoặc các mẫu rất giống nhau, mô hình có thể "học vẹt" các phản hồi cụ thể cho các câu hỏi đó thay vì học cách tổng quát hóa và hiểu ngữ cảnh. Điều này dẫn đến việc mô hình hoạt động kém khi gặp các câu hỏi mới, chưa từng thấy trong dataset huấn luyện.

### 2.2. Cân bằng Độ dài Phản hồi

Các mô hình ngôn ngữ có xu hướng bắt chước phong cách và độ dài của dữ liệu mà chúng được huấn luyện. Nếu dataset fine-tuning chứa các phản hồi của trợ lý quá dài so với câu hỏi của người dùng, mô hình có thể học cách tạo ra các phản hồi dài dòng, không cần thiết, ngay cả khi câu hỏi chỉ yêu cầu một câu trả lời ngắn gọn. Điều này làm giảm tính hiệu quả và trải nghiệm người dùng.

### 2.3. Tăng cường Tính Đa dạng và Mạnh mẽ

Một dataset với các câu hỏi người dùng hạn chế về cách diễn đạt có thể khiến mô hình chỉ hoạt động tốt trong các tình huống cụ thể đó. Bằng cách tăng cường dữ liệu (data augmentation) – tạo ra nhiều biến thể của cùng một câu hỏi – chúng ta giúp mô hình tiếp xúc với nhiều cách diễn đạt khác nhau cho cùng một ý định, từ đó nâng cao khả năng hiểu và phản hồi của mô hình trong thế giới thực.

### 2.4. Tối ưu hóa Hiệu quả Huấn luyện

Dữ liệu trùng lặp không chỉ gây hại cho chất lượng mô hình mà còn lãng phí tài nguyên tính toán. Việc huấn luyện trên một dataset sạch, được tối ưu hóa sẽ giúp quá trình fine-tuning diễn ra nhanh hơn và hiệu quả hơn, giảm chi phí và thời gian cần thiết.

## 3. Định dạng Dataset JSONL

Trước khi đi sâu vào các bước tối ưu hóa, điều quan trọng là phải hiểu định dạng JSONL mà chúng ta đang làm việc. JSONL (JSON Lines) là một định dạng phổ biến cho dữ liệu có cấu trúc, trong đó mỗi dòng là một đối tượng JSON hợp lệ. Đối với fine-tuning mô hình ngôn ngữ, mỗi đối tượng JSON thường đại diện cho một cuộc hội thoại hoặc một cặp hỏi-đáp. Cấu trúc phổ biến nhất là một đối tượng JSON chứa một khóa `"messages"` là một danh sách các tin nhắn, mỗi tin nhắn có `"role"` (ví dụ: `"user"`, `"assistant"`, `"system"`) và `"content"`.

**Ví dụ về một mục JSONL:**

```json
{"messages": [{"role": "user", "content": "Hãy giải thích về Clean Architecture"}, {"role": "assistant", "content": "Clean Architecture là một triết lý thiết kế phần mềm nhằm tạo ra các hệ thống dễ kiểm thử, dễ bảo trì và độc lập với các công nghệ bên ngoài..."}]}
```

## 4. Các Bước Tối ưu hóa Dataset

Chúng ta sẽ thực hiện các bước tối ưu hóa theo trình tự để đảm bảo hiệu quả cao nhất.

### 4.1. Bước 1: Loại bỏ các mục trùng lặp

Việc loại bỏ các bản ghi trùng lặp là bước đầu tiên và cơ bản nhất để làm sạch dataset. Dữ liệu trùng lặp không mang lại giá trị gia tăng cho quá trình huấn luyện mà còn có thể gây ra hiện tượng "học vẹt" và lãng phí tài nguyên.

#### 4.1.1. Tại sao cần loại bỏ trùng lặp?

-   **Giảm "học vẹt":** Mô hình có thể ghi nhớ các phản hồi cụ thể cho các câu hỏi trùng lặp thay vì học các nguyên tắc tổng quát.
-   **Tăng hiệu quả huấn luyện:** Giảm kích thước dataset, từ đó giảm thời gian và chi phí huấn luyện.
-   **Cải thiện khả năng tổng quát hóa:** Khuyến khích mô hình học từ các mẫu đa dạng hơn.

#### 4.1.2. Cách thực hiện

Chúng ta sẽ đọc từng dòng trong tệp JSONL, chuyển đổi mỗi đối tượng JSON thành một chuỗi duy nhất (để có thể so sánh và lưu vào một `set`), và chỉ giữ lại các bản ghi duy nhất. Điều quan trọng là phải chuẩn hóa chuỗi JSON (ví dụ: sắp xếp các khóa) để đảm bảo rằng các đối tượng JSON có cùng nội dung nhưng thứ tự khóa khác nhau vẫn được coi là trùng lặp.

**Mã Python để loại bỏ trùng lặp:**

```python
import json

def remove_duplicates(input_file, output_file):
    unique_entries = set()
    cleaned_data = []
    duplicate_count = 0

    with open(input_file, 'r', encoding='utf-8') as infile:
        for line in infile:
            try:
                data = json.loads(line)
                # Convert the dictionary to a JSON string to use it in a set
                # ensure_ascii=False to handle Vietnamese characters correctly
                # sort_keys=True to ensure consistent order for comparison
                entry_str = json.dumps(data, ensure_ascii=False, sort_keys=True)
                if entry_str not in unique_entries:
                    unique_entries.add(entry_str)
                    cleaned_data.append(line.strip())
                else:
                    duplicate_count += 1
            except json.JSONDecodeError as e:
                print(f"Lỗi giải mã JSON: {e} trong dòng: {line.strip()}")

    with open(output_file, 'w', encoding='utf-8') as outfile:
        for line in cleaned_data:
            outfile.write(line + '\n')

    print(f"Số mục ban đầu: {len(unique_entries) + duplicate_count}. Số mục sau khi xóa trùng lặp: {len(cleaned_data)}. Số mục trùng lặp đã xóa: {duplicate_count}")

if __name__ == "__main__":
    input_jsonl_path = "/home/ubuntu/upload/typeorm_nestjs_dataset.jsonl" # Thay đổi đường dẫn đến tệp của bạn
    output_jsonl_path = "/home/ubuntu/typeorm_nestjs_dataset_cleaned.jsonl"
    remove_duplicates(input_jsonl_path, output_jsonl_path)
```

**Giải thích mã:**

-   `unique_entries`: Một `set` để lưu trữ các chuỗi JSON đã chuẩn hóa của các mục duy nhất.
-   `cleaned_data`: Một danh sách để lưu trữ các dòng JSONL đã được làm sạch.
-   Mỗi dòng được đọc, phân tích cú pháp thành đối tượng JSON, sau đó được chuyển đổi lại thành chuỗi JSON với `sort_keys=True` để đảm bảo tính nhất quán khi so sánh. `ensure_ascii=False` được sử dụng để giữ nguyên các ký tự tiếng Việt.
-   Nếu chuỗi JSON của mục hiện tại chưa có trong `unique_entries`, nó sẽ được thêm vào `set` và dòng gốc được thêm vào `cleaned_data`.
-   Cuối cùng, các dòng đã làm sạch được ghi vào tệp đầu ra mới.

#### 4.1.3. Kết quả mong đợi

Sau khi chạy script này, bạn sẽ có một tệp JSONL mới (`typeorm_nestjs_dataset_cleaned.jsonl`) không còn các bản ghi trùng lặp. Script sẽ in ra số lượng mục ban đầu, số mục sau khi xóa trùng lặp và số mục trùng lặp đã xóa.

### 4.2. Bước 2: Xử lý các câu trả lời quá dài

Các phản hồi của trợ lý có độ dài cực lớn có thể gây ra nhiều vấn đề trong quá trình fine-tuning. Mô hình có thể học cách tạo ra các phản hồi dài dòng không cần thiết, hoặc gặp khó khăn trong việc tập trung vào các thông tin quan trọng. Có hai chiến lược chính để xử lý vấn đề này: chia nhỏ (splitting) hoặc tóm tắt (summarization). Trong hướng dẫn này, chúng ta sẽ tập trung vào kỹ thuật chia nhỏ, vì nó giữ lại toàn bộ thông tin gốc và tạo ra nhiều cặp hỏi-đáp hơn, làm tăng kích thước dataset.

#### 4.2.1. Tại sao cần xử lý các phản hồi dài?

-   **Kiểm soát độ dài phản hồi:** Đảm bảo mô hình tạo ra các phản hồi có độ dài phù hợp với ngữ cảnh.
-   **Cải thiện khả năng học:** Mô hình có thể tập trung tốt hơn vào các phần nhỏ hơn của thông tin.
-   **Tăng số lượng mẫu huấn luyện:** Chia nhỏ một phản hồi dài thành nhiều cặp hỏi-đáp nhỏ hơn sẽ làm tăng số lượng mẫu trong dataset, giúp mô hình học được nhiều mối quan hệ hơn giữa câu hỏi và phản hồi.

#### 4.2.2. Cách thực hiện (Chia nhỏ)

Chúng ta sẽ xác định một ngưỡng độ dài tối đa cho các phản hồi của trợ lý. Nếu một phản hồi vượt quá ngưỡng này, nó sẽ được chia thành nhiều phần nhỏ hơn. Mỗi phần sẽ được ghép nối với câu hỏi gốc của người dùng. Để duy trì tính liên tục của cuộc hội thoại, các câu hỏi cho các phần tiếp theo có thể được sửa đổi để chỉ ra rằng chúng là phần tiếp theo của một chủ đề lớn hơn (ví dụ: "Tiếp theo về: [câu hỏi gốc]").

**Mã Python để xử lý các phản hồi dài:**

```python
import json
import re

def split_long_response(response_text, max_length=2000, min_split_length=500):
    # Attempt to split by major headings or logical sections
    sections = re.split(r'\n\n### .*?\n\n|\n\n## .*?\n\n|\n\n#### .*?\n\n', response_text)
    
    # Filter out empty strings and strip whitespace from sections
    sections = [s.strip() for s in sections if s.strip()]

    if len(response_text) <= max_length:
        return [response_text]

    # If splitting by headings doesn't work well or results in too many small parts
    # try splitting by paragraphs or sentences
    if len(sections) == 1 or any(len(s) < min_split_length for s in sections):
        # Fallback to splitting by paragraphs if sections are too short or only one section
        paragraphs = [p.strip() for p in response_text.split('\n\n') if p.strip()]
        
        # If still too long, split by sentences
        if not paragraphs or any(len(p) > max_length for p in paragraphs):
            sentences = re.split(r'(?<=[.!?])\s+', response_text)
            current_chunk = []
            chunks = []
            current_length = 0

            for sentence in sentences:
                if current_length + len(sentence) + 1 > max_length and current_chunk:
                    chunks.append(' '.join(current_chunk))
                    current_chunk = [sentence]
                    current_length = len(sentence)
                else:
                    current_chunk.append(sentence)
                    current_length += len(sentence) + 1
            if current_chunk:
                chunks.append(' '.join(current_chunk))
            return chunks
        else:
            return paragraphs
    else:
        return sections

def process_dataset(input_file, output_file, max_assistant_length=4000):
    new_entries = []
    with open(input_file, 'r', encoding='utf-8') as f:
        for line in f:
            try:
                entry = json.loads(line)
                messages = entry.get('messages', [])

                user_message = None
                assistant_response = None

                # Find the last user message and assistant response
                for i in range(len(messages) - 1, -1, -1):
                    if messages[i]['role'] == 'assistant':
                        assistant_response = messages[i]['content']
                    elif messages[i]['role'] == 'user' and assistant_response is not None:
                        user_message = messages[i]['content']
                        break
                
                if user_message and assistant_response and len(assistant_response) > max_assistant_length:
                    # Split the long assistant response
                    split_responses = split_long_response(assistant_response, max_length=max_assistant_length)
                    
                    # Create new entries for each split response
                    for i, part in enumerate(split_responses):
                        new_entry = {"messages": [
                            {"role": "user", "content": user_message},
                            {"role": "assistant", "content": part}
                        ]}
                        # If it's not the first part, modify the user message to indicate continuation
                        if i > 0:
                            new_entry["messages"][0]["content"] = f"Tiếp theo về: {user_message}"
                        new_entries.append(new_entry)
                else:
                    new_entries.append(entry)

            except json.JSONDecodeError as e:
                print(f"Lỗi giải mã JSON: {e} trong dòng: {line.strip()}")

    with open(output_file, 'w', encoding='utf-8') as f:
        for entry in new_entries:
            f.write(json.dumps(entry, ensure_ascii=False) + '\n')

    print(f"Đã xử lý các câu trả lời quá dài. Tổng số mục mới: {len(new_entries)}")

if __name__ == "__main__":
    input_file = "/home/ubuntu/typeorm_nestjs_dataset_cleaned.jsonl" # Sử dụng tệp đã xóa trùng lặp
    output_file = "/home/ubuntu/typeorm_nestjs_dataset_processed.jsonl"
    process_dataset(input_file, output_file)
```

**Giải thích mã:**

-   Hàm `split_long_response` cố gắng chia phản hồi dựa trên các tiêu đề Markdown (`###`, `##`, `####`) trước. Nếu không hiệu quả, nó sẽ chuyển sang chia theo đoạn văn, và cuối cùng là theo câu nếu các đoạn văn vẫn quá dài.
-   Hàm `process_dataset` đọc từng mục, kiểm tra độ dài của phản hồi trợ lý. Nếu vượt quá `max_assistant_length` (mặc định 4000 ký tự), nó sẽ gọi `split_long_response`.
-   Mỗi phần được chia nhỏ sẽ tạo thành một mục mới trong dataset. Đối với các phần tiếp theo, câu hỏi của người dùng được điều chỉnh để duy trì ngữ cảnh (ví dụ: "Tiếp theo về: [câu hỏi gốc]").

#### 4.2.3. Kết quả mong đợi

Sau bước này, dataset của bạn sẽ có nhiều mục hơn, và độ dài trung bình của các phản hồi trợ lý sẽ giảm xuống, nằm trong ngưỡng hợp lý. Điều này giúp mô hình học cách tạo ra các phản hồi có độ dài phù hợp hơn.

### 4.3. Bước 3: Tăng cường Câu hỏi Người dùng (Data Augmentation)

Để làm cho mô hình mạnh mẽ hơn và có khả năng tổng quát hóa tốt hơn, chúng ta cần tăng cường sự đa dạng của các câu hỏi người dùng. Điều này giúp mô hình hiểu cùng một ý định được diễn đạt theo nhiều cách khác nhau.

#### 4.3.1. Tại sao cần tăng cường câu hỏi?

-   **Cải thiện khả năng hiểu ý định:** Mô hình sẽ học cách nhận diện cùng một ý định ngay cả khi câu hỏi được diễn đạt khác nhau.
-   **Tăng cường khả năng tổng quát hóa:** Giúp mô hình hoạt động tốt hơn trên các câu hỏi thực tế mà nó chưa từng thấy.
-   **Mở rộng phạm vi ứng dụng:** Mô hình có thể xử lý nhiều loại câu hỏi hơn, từ đó tăng tính linh hoạt.

#### 4.3.2. Cách thực hiện

Đối với mỗi câu hỏi người dùng trong dataset, chúng ta sẽ tạo ra một số biến thể khác nhau. Các biến thể này có thể bao gồm việc sử dụng từ đồng nghĩa, thay đổi cấu trúc câu, hoặc thêm các cụm từ bổ sung mà không làm thay đổi ý nghĩa cốt lõi của câu hỏi. Mỗi biến thể câu hỏi sẽ được ghép nối với phản hồi trợ lý tương ứng để tạo thành một mục mới trong dataset.

**Mã Python để tăng cường câu hỏi người dùng:**

```python
import json

def augment_question(question):
    variations = [
        question, # Original question
        f"Bạn có thể giải thích thêm về {question.lower().replace('hãy giải thích về', '').strip()} không?",
        f"Thông tin chi tiết về {question.lower().replace('hãy giải thích về', '').strip()} là gì?",
        f"Kể cho tôi nghe về {question.lower().replace('hãy giải thích về', '').strip()}.",
        f"Tôi muốn tìm hiểu về {question.lower().replace('hãy giải thích về', '').strip()}.",
        f"Định nghĩa của {question.lower().replace('hãy giải thích về', '').strip()} là gì?",
        f"Giải thích rõ hơn về {question.lower().replace('hãy giải thích về', '').strip()}.",
        f"Xin vui lòng trình bày về {question.lower().replace('hãy giải thích về', '').strip()}."
    ]
    return list(set(variations)) # Return unique variations

def augment_dataset(input_file, output_file):
    augmented_entries = []
    with open(input_file, 'r', encoding='utf-8') as f:
        for line in f:
            try:
                entry = json.loads(line)
                messages = entry.get('messages', [])

                user_message = None
                assistant_response = None

                # Find the last user message and assistant response
                for i in range(len(messages) - 1, -1, -1):
                    if messages[i]['role'] == 'assistant':
                        assistant_response = messages[i]['content']
                    elif messages[i]['role'] == 'user' and assistant_response is not None:
                        user_message = messages[i]['content']
                        break
                
                if user_message and assistant_response:
                    augmented_questions = augment_question(user_message)
                    for q in augmented_questions:
                        new_entry = {"messages": [
                            {"role": "user", "content": q},
                            {"role": "assistant", "content": assistant_response}
                        ]}
                        augmented_entries.append(new_entry)
                else:
                    augmented_entries.append(entry)

            except json.JSONDecodeError as e:
                print(f"Lỗi giải mã JSON: {e} trong dòng: {line.strip()}")

    with open(output_file, 'w', encoding='utf-8') as f:
        for entry in augmented_entries:
            f.write(json.dumps(entry, ensure_ascii=False) + '\n')

    print(f"Đã tăng cường dataset. Tổng số mục mới: {len(augmented_entries)}")

if __name__ == "__main__":
    input_file = "/home/ubuntu/typeorm_nestjs_dataset_processed.jsonl" # Sử dụng tệp đã xử lý câu trả lời dài
    output_file = "/home/ubuntu/typeorm_nestjs_dataset_augmented.jsonl"
    augment_dataset(input_file, output_file)
```

**Giải thích mã:**

-   Hàm `augment_question` nhận một câu hỏi và trả về một danh sách các biến thể của câu hỏi đó. Các biến thể được tạo ra bằng cách thay thế các cụm từ thông dụng như "Hãy giải thích về" bằng các cụm từ khác có ý nghĩa tương tự.
-   Hàm `augment_dataset` đọc từng mục, trích xuất câu hỏi người dùng và phản hồi trợ lý. Sau đó, nó tạo ra các biến thể câu hỏi và tạo các mục mới trong dataset cho mỗi biến thể, ghép nối chúng với phản hồi trợ lý gốc.

#### 4.3.3. Kết quả mong đợi

Sau bước này, dataset của bạn sẽ tăng lên đáng kể về số lượng mẫu. Quan trọng hơn, các câu hỏi người dùng sẽ đa dạng hơn về cách diễn đạt, giúp mô hình học được sự linh hoạt trong việc hiểu ý định của người dùng.

### 4.4. Bước 4: Loại bỏ trùng lặp lần cuối (Sau Data Augmentation)

Sau khi tăng cường dữ liệu, có khả năng một số mục trùng lặp mới có thể phát sinh do các biến thể câu hỏi hoặc sự kết hợp giữa câu hỏi và phản hồi. Do đó, việc chạy lại bước loại bỏ trùng lặp là cần thiết để đảm bảo dataset cuối cùng là hoàn toàn sạch.

#### 4.4.1. Tại sao cần loại bỏ trùng lặp lần cuối?

-   **Đảm bảo tính duy nhất:** Ngăn chặn các bản ghi trùng lặp mới xuất hiện sau quá trình tăng cường dữ liệu.
-   **Duy trì chất lượng dataset:** Giữ cho dataset tinh gọn và hiệu quả cho fine-tuning.

#### 4.4.2. Cách thực hiện

Sử dụng lại script loại bỏ trùng lặp từ Bước 1, nhưng áp dụng nó cho tệp dataset đã được tăng cường.

**Mã Python để loại bỏ trùng lặp lần cuối:**

```python
import json

def remove_duplicates(input_file, output_file):
    unique_entries = set()
    cleaned_data = []
    duplicate_count = 0

    with open(input_file, 'r', encoding='utf-8') as infile:
        for line in infile:
            try:
                data = json.loads(line)
                entry_str = json.dumps(data, ensure_ascii=False, sort_keys=True)
                if entry_str not in unique_entries:
                    unique_entries.add(entry_str)
                    cleaned_data.append(line.strip())
                else:
                    duplicate_count += 1
            except json.JSONDecodeError as e:
                print(f"Lỗi giải mã JSON: {e} trong dòng: {line.strip()}")

    with open(output_file, 'w', encoding='utf-8') as outfile:
        for line in cleaned_data:
            outfile.write(line + '\n')

    print(f"Số mục ban đầu: {len(unique_entries) + duplicate_count}. Số mục sau khi xóa trùng lặp: {len(cleaned_data)}. Số mục trùng lặp đã xóa: {duplicate_count}")

if __name__ == "__main__":
    input_jsonl_path = "/home/ubuntu/typeorm_nestjs_dataset_augmented.jsonl" # Sử dụng tệp đã tăng cường
    output_jsonl_path = "/home/ubuntu/typeorm_nestjs_dataset_final.jsonl"
    remove_duplicates(input_jsonl_path, output_jsonl_path)
```

**Giải thích mã:**

-   Mã này giống hệt với mã loại bỏ trùng lặp ở Bước 1, nhưng đầu vào là tệp đã được tăng cường (`typeorm_nestjs_dataset_augmented.jsonl`).

#### 4.4.3. Kết quả mong đợi

Dataset cuối cùng (`typeorm_nestjs_dataset_final.jsonl`) sẽ là phiên bản sạch nhất, không có trùng lặp và đã được tối ưu hóa về độ dài phản hồi cũng như sự đa dạng của câu hỏi. Đây là tệp bạn nên sử dụng cho quá trình fine-tuning mô hình của mình.

## 5. Phân tích Chất lượng Dataset Sau Tối ưu hóa

Sau khi hoàn thành tất cả các bước tối ưu hóa, việc phân tích lại dataset cuối cùng là rất quan trọng để xác nhận rằng các cải tiến đã được thực hiện thành công. Bạn có thể sử dụng một script tương tự như script phân tích ban đầu để kiểm tra các số liệu thống kê như tổng số mục, số lượng trùng lặp, và phân phối độ dài tin nhắn.

**Mã Python để phân tích chất lượng dataset cuối cùng:**

```python
import json

def analyze_dataset_quality(file_path):
    total_entries = 0
    total_messages = 0
    user_message_lengths = []
    assistant_message_lengths = []
    unique_user_prompts = set()
    duplicate_entries = 0

    with open(file_path, 'r', encoding='utf-8') as f:
        for i, line in enumerate(f):
            total_entries += 1
            try:
                data = json.loads(line)
                messages = data.get('messages', [])
                total_messages += len(messages)

                entry_str = json.dumps(data, ensure_ascii=False, sort_keys=True)
                if entry_str in unique_user_prompts:
                    duplicate_entries += 1
                else:
                    unique_user_prompts.add(entry_str)

                for message in messages:
                    role = message.get('role')
                    content = message.get('content', '')

                    if role == 'user':
                        user_message_lengths.append(len(content))
                    elif role == 'assistant':
                        assistant_message_lengths.append(len(content))

            except json.JSONDecodeError:
                pass # Already handled by previous check

    print(f"Tổng số mục trong dataset: {total_entries}")
    print(f"Tổng số tin nhắn (user + assistant): {total_messages}")
    print(f"Số lượng mục trùng lặp: {duplicate_entries}")

    if user_message_lengths:
        print(f"Độ dài trung bình tin nhắn người dùng: {sum(user_message_lengths) / len(user_message_lengths):.2f} ký tự")
        print(f"Độ dài tối thiểu tin nhắn người dùng: {min(user_message_lengths)} ký tự")
        print(f"Độ dài tối đa tin nhắn người dùng: {max(user_message_lengths)} ký tự")
    else:
        print("Không có tin nhắn người dùng nào được tìm thấy.")

    if assistant_message_lengths:
        print(f"Độ dài trung bình tin nhắn trợ lý: {sum(assistant_message_lengths) / len(assistant_message_lengths):.2f} ký tự")
        print(f"Độ dài tối thiểu tin nhắn trợ lý: {min(assistant_message_lengths)} ký tự")
        print(f"Độ dài tối đa tin nhắn trợ lý: {max(assistant_message_lengths)} ký tự")
    else:
        print("Không có tin nhắn trợ lý nào được tìm thấy.")

if __name__ == "__main__":
    analyze_dataset_quality("/home/ubuntu/typeorm_nestjs_dataset_final.jsonl") # Sử dụng tệp cuối cùng
```

**Kết quả phân tích dự kiến (ví dụ):**

| Chỉ số Phân tích             | Dataset Ban đầu | Dataset Sau Tối ưu hóa |
| :-------------------------- | :-------------- | :--------------------- |
| Tổng số mục                 | 133             | 10352                  |
| Số lượng mục trùng lặp      | 2               | 0                      |
| Độ dài TB tin nhắn người dùng | ~46 ký tự       | ~67 ký tự              |
| Độ dài TB tin nhắn trợ lý   | ~3033 ký tự     | ~306 ký tự             |
| Độ dài tối đa tin nhắn trợ lý | ~50874 ký tự    | ~11428 ký tự           |

Bảng trên cho thấy sự cải thiện rõ rệt về số lượng mẫu, loại bỏ trùng lặp, và cân bằng độ dài tin nhắn, đặc biệt là giảm đáng kể độ dài trung bình và tối đa của phản hồi trợ lý.

## 6. Kết luận

Việc tối ưu hóa dataset JSONL là một bước quan trọng để đảm bảo quá trình fine-tuning mô hình ngôn ngữ đạt được hiệu quả cao nhất. Bằng cách loại bỏ trùng lặp, xử lý các phản hồi quá dài, và tăng cường sự đa dạng của câu hỏi người dùng, bạn có thể tạo ra một dataset chất lượng cao, giúp mô hình học hiệu quả hơn, tổng quát hóa tốt hơn và đưa ra các phản hồi phù hợp, tự nhiên hơn.

Dataset cuối cùng, `typeorm_nestjs_dataset_final.jsonl`, là kết quả của quá trình tối ưu hóa này và sẵn sàng để được sử dụng cho việc fine-tuning mô hình Mistral hoặc bất kỳ mô hình ngôn ngữ nào khác.

## 7. Tài liệu tham khảo

[1] Robert C. Martin. (2017). *Clean Architecture: A Craftsman's Guide to Software Structure and Design*. Prentice Hall. (Tham khảo cho khái niệm Clean Architecture)

[2] Eric Evans. (2003). *Domain-Driven Design: Tackling Complexity in the Heart of Software*. Addison-Wesley Professional. (Tham khảo cho khái niệm Domain-Driven Design)

[3] Sam Newman. (2015). *Building Microservices: Designing Fine-Grained Systems*. O'Reilly Media. (Tham khảo cho khái niệm Microservices)

[4] Jason Brownlee. (2019). *A Gentle Introduction to Data Augmentation for Deep Learning*. Machine Learning Mastery. [https://machinelearningmastery.com/data-augmentation-for-deep-learning/](https://machinelearningmastery.com/data-augmentation-for-deep-learning/)

[5] OpenAI. (n.d.). *Fine-tuning*. OpenAI Documentation. [https://platform.openai.com/docs/guides/fine-tuning](https://platform.openai.com/docs/guides/fine-tuning)


