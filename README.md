import sys  
from collections import Counter  
import re  
import threading  
from concurrent.futures import ThreadPoolExecutor  
  
# 假设的停用词列表  
STOPWORDS = set([  
    'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 'your',  
    'yours', 'yourself', 'yourselves', 'he', 'him', 'his', 'himself', 'she',  
    'her', 'hers', 'herself', 'it', 'its', 'itself', 'they', 'them', 'their',  
    'theirs', 'themselves', 'what', 'which', 'who', 'whom', 'this', 'that',  
    'these', 'those', 'am', 'is', 'are', 'was', 'were', 'be', 'been', 'being',  
    'have', 'has', 'had', 'having', 'do', 'does', 'did', 'doing', 'a', 'an',  
    'the', 'and', 'but', 'if', 'or', 'because', 'as', 'until', 'while', 'of',  
    'at', 'by', 'for', 'with', 'about', 'against', 'between', 'into', 'through',  
    'during', 'before', 'after', 'above', 'below', 'to', 'from', 'up', 'down',  
    'in', 'out', 'on', 'off', 'over', 'under', 'again', 'further', 'then',  
    'once', 'here', 'there', 'when', 'where', 'why', 'how', 'all', 'any',  
    'both', 'each', 'few', 'more', 'most', 'other', 'some', 'such', 'no',  
    'nor', 'not', 'only', 'own', 'same', 'so', 'than', 'too', 'very',  
    's', 't', 'can', 'will', 'just', 'don', 'should', 'now'  
])  
  
def read_file(file_path, is_stdin=False):  
    """从文件或标准输入读取内容，并返回字符串形式的内容"""  
    if is_stdin:  
        return sys.stdin.read()  
    try:  
        with open(file_path, 'r', encoding='utf-8') as file:  
            return file.read()  
    except FileNotFoundError:  
        print(f"Error: 文件 {file_path} 未找到。")  
        sys.exit(1)  
    except Exception as e:  
        print(f"读取文件时发生错误: {e}")  
        sys.exit(1)  
  
def process_text(text):  
    """处理文本，返回单词计数的Counter对象"""  
    # 转换为小写并分割成单词列表  
    words = re.findall(r'\b\w+\b', text.lower())  
    # 移除停用词和标点（这里停用词已在前面移除）  
    words = [word for word in words if word not in STOPWORDS]  
    # 计算单词频率  
    return Counter(words)  
  
def process_files(file_paths):  
    """并行处理多个文件"""  
    with ThreadPoolExecutor(max_workers=5) as executor:  
        future_to_file = {executor.submit(process_single_file, f): f for f in file_paths}  
        for future in concurrent.futures.as_completed(future_to_file):  
            file_path, word_counts = future.result()  
            print(f"处理完成: {file_path}")  
            # 这里可以进一步处理或合并word_counts  
  
def process_single_file(file_path):  
    """处理单个文件并返回单词计数"""  
    text = read_file(file_path)  
    word_counts = process_text(text)  
    return file_path, word_counts  
  
def main():  
    if len(sys.argv) < 2:  
        print("使用方法: python script.py 文件路径1 [文件路径2 ...]")  
        print("或: python script.py - (从标准输入读取)")  
        sys.exit(1)  
  
    file_paths = []  
    for arg in sys.argv[1:]:  
        if arg == '-':  
            file_paths.append(arg)  # 标记为从标准输入读取  
        else:  
            file_paths.append(arg)  
  
    if '-' in file_paths:  
        # 如果包含从标准输入读取的指示，则单独处理  
        word_counts = process_single_file('-')  
        print_top_words(word_counts)  
    else:  
        # 否则，并行处理所有文件  
        process_files(file_paths)  
  
def print_top_words(word_counts, top_n=10):  
    """打印频率最高的前top_n个单词"""  
    for word, count in word_counts.most_common(top_n):  
        print(f"{word}: {count}")  
  
if __name__ == "__main__":  
    main()  
  
# 注意：上述代码中的process_files函数仅用于展示如何并行处理多个文件，  
# 但它没有实际打印每个文件的单词频率。在实际应用中，您可能需要  
# 修改它以合并或分别处理每个文件的单词计数.
