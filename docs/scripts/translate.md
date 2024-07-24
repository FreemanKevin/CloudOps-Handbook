## 繁體轉簡體
```python
from opencc import OpenCC

def convert_t2s(input_file, output_file):
    cc = OpenCC('t2s')  # 繁体到简体
    try:
        with open(input_file, 'r', encoding='utf-8') as file:
            traditional_content = file.read()
            print("读取文件成功")
            simplified_content = cc.convert(traditional_content)
            print("转换成功")
        
        with open(output_file, 'w', encoding='utf-8') as file:
            file.write(simplified_content)
            print("写入文件成功")
    except Exception as e:
        print("发生错误:", e)

# 调用函数，输入文件和输出文件
convert_t2s('input.md', 'output.md')
```