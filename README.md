特别鸣谢医工
1、找到需要获取的地区的地址列表，这个地址列表基本是固定不变的。如自家有开通iptv，可以通过抓包获取本地的地址。

2、设定要查询的省份，通过关键字在fofa上查询出组播的IP，结果存至变量result_urls

search_url = 'https://fofa.info/result?qbase64='
search_txt = f'\"udpxy\" && country=\"CN\" && region=\"{province}\"'
# 将字符串编码为字节流
bytes_string = search_txt.encode('utf-8')
# 使用 base64 进行编码
search_txt = base64.b64encode(bytes_string).decode('utf-8')
search_url += search_txt
print(f"{current_time} province : {province}，search_url : {search_url}")
response = requests.get(search_url, timeout=30)
# 处理响应
response.raise_for_status()
# 检查请求是否成功
html_content = response.text
# 使用BeautifulSoup解析网页内容
html_soup = BeautifulSoup(html_content, "html.parser")
# 查找所有符合指定格式的网址
# 设置匹配的格式，如http://8.8.8.8:8888
pattern = r"http://\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}:\d+"
urls_all = re.findall(pattern, html_content)
# 去重得到唯一的URL列表
result_urls = set(urls_all)
3、通过cv2测试组播IP是否可用：测试视频链接(组播result_urls+其中一个节目的组播地址urls_udp）分辨率是否大于0，大于0的组播IP结果保存至res.txt

#对应省份的组播地址:重庆联通cctv1：225.0.4.74:7980，重庆电信cctv1:235.254.199.51:7980，广东电信广东卫视239.77.1.19:5146
urls_udp = "/udp/239.77.1.19:5146"
valid_ips = []
# 遍历所有视频链接
for url in result_urls:
    ip_port = url.replace("http://", "")
    video_url = url + urls_udp

    # 用OpenCV读取视频
    cap = cv2.VideoCapture(video_url)

    # 检查视频是否成功打开
    if not cap.isOpened():
        print(f"{current_time} {video_url} 无效")
    else:
        # 读取视频的宽度和高度
        width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
        height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
        print(f"{current_time} {video_url} 的分辨率为 {width}x{height}")
        # 检查分辨率是否大于0
        if width > 0 and height > 0:
            valid_ips.append(ip_port)
        # 关闭视频流
        cap.release()

# 将分辨率大于0的结果保存到res.txt文件中
with open("res.txt", "w") as file:
    for ip in valid_ips:
        file.write(ip + "\n")
4、制作节目列表，用http://{ip}/udp/替rtp中的rtp://，结果保存至我们需要的iptv.txt，

with open('rtp.txt', 'r', encoding='utf-8') as file:
    data = file.read()

with open('iptv.txt', 'w') as new_file:
    for ip in valid_ips:
        new_data = data.replace("rtp://", f"http://{ip}/udp/")
        new_file.write(new_data)

print("已生成播放列表，保存至iptv.txt")
5、同时，我们可以将iptv.txt转成带台标和节目列表的iptv.m3u



二、测试代码

我们以获取广东电信为例：

1、我们将网上找到的广东电信组播地址保存至rtp.txt

2、修改代码参数，打开udp_iptv.py，

修改province_names 为需要查询的省份，广东

修改对应省份的组播地址urls_udp的值，我们需要修改为其中一个节目地址就可以，如我们填入广东电信广东卫视的地址239.77.1.19:5146


说明∶


1、我们第一步都是找到需要获取省份的组播地址列表复制到rtp.txt，然后修改udp_iptv.py的代码参数或者运行udp_iptv2.py按提示输入参数。

2、所需运行库：requests，bs4，cv2


安装运行库：pip install 所需库 -i 国内源或pip3 install 所需库 -i 国内源

pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install opencv-python -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install Beautifulsoup4 -i https://pypi.tuna.tsinghua.edu.cn/simple


3、run udp_iptv.py

4、扩展应用∶我们可以每天定期运行一次udp_iptv.py，播放器播放地址指向我们的iptv.txt或iptv.m3u，这样就可以不用等失效了手动运行更新了。

三、相关下载

1、文件内容：

运行所需文件：udp_iptv.py、trp/对应省份_运营商.txt

生成文件：iptv.txt、iptv.m3u


2、说明：组目如果连接数较多可能会卡顿,fofa测绘站每个ip每天有请求次数限制，不要频繁请求

