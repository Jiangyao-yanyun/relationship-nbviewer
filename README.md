import networkx as nx
from pyvis.network import Network
import ipywidgets as widgets
from IPython.display import display, HTML
import json

# 定义关系数据
relations = {
    "江杳": {
        "徒弟": ["轩辕慕玥", "星枫飘缈", "鱼幼曦", "剑起御春风（已叛逃师门）"],
        "侠缘": ["江音"]
    },
    "江音": {
        "徒弟": ["潇湘妃"]
    },
    "鱼幼曦": {
        "徒弟": ["藤椒手枪腿", "沝仙", "莫许秋安", "月云湛", "沐泠秋"],
        "侠缘": ["遥知不归处"]
    },
    "莫许秋安": {
        "徒弟": ["封存余味", "白夜云帆"]
    },
    "遥知不归处": {
        "徒弟": ["林酒行", "禾老三", "龙弶", "流年"]
    },
    "龙弶": {
        "徒弟": ["凤鹤川（已叛逃师门）", "江嬴晚", "陆炎墨"]
    },
    "剑起御春风": {
        "徒弟": ["菠萝油多士", "木白衍", "寒岁聿"]
    },
    "寒岁聿": {
        "徒弟": ["玺安衍", "陆诩晨", "上头了呀"],
        "侠缘": ["木白衍"]
    },
    "上头了呀": {
        "徒弟": ["段慕柔", "姜晚玥", "吉吉国王", "画依依", "只喝百事呗"]
    },
    "江月": {
        "徒弟": ["机韶逸（疑似已被夺舍）", "小宝塔", "老六", "无敌小呆呆"],
        "侠缘": ["临渊羡月"],
        "姐妹": ["晏燕儿"]
    },
    "晏燕儿": {
        "徒弟": ["奶味水饺", "君澜澜"]
    },
    "机韶逸": {
        "徒弟": ["殷小闲"],
        "侠缘": ["江嬴晚"]
    },
    "沐泠秋": {
        "徒弟": ["緍缘", "红莲嬅"]
    },
    "禾老三": {
        "徒弟": ["悦清言"]
    },
    "鹿楚盼": {
        "徒弟": ["李佳盛"]
    }
}

# 创建关系图
G = nx.Graph()

# 添加节点和关系
for person, details in relations.items():
    # 添加人物节点
    G.add_node(person, title=person, group="人物")
    
    # 添加关系
    for relation_type, related_people in details.items():
        for related_person in related_people:
            # 处理特殊标记
            if "（" in related_person:
                display_name = related_person.split("（")[0]
                note = related_person.split("（")[1].replace("）", "")
            else:
                display_name = related_person
                note = ""
                
            # 添加关系节点
            G.add_node(display_name, title=display_name, group="人物", note=note)
            
            # 添加关系边
            if relation_type == "徒弟":
                G.add_edge(person, display_name, title="师徒", color="#FF5733", width=3)
            elif relation_type == "侠缘":
                G.add_edge(person, display_name, title="侠缘", color="#33FF57", width=3, dashes=True)
            elif relation_type == "姐妹":
                G.add_edge(person, display_name, title="姐妹", color="#FF33A8", width=3)
            else:
                G.add_edge(person, display_name, title=relation_type, color="#33A8FF", width=3)

# 创建网络可视化
net = Network(height="750px", width="100%", notebook=True, bgcolor="#222222", font_color="white")
net.from_nx(G)

# 设置节点样式
for node in net.nodes:
    node["size"] = 25
    node["borderWidth"] = 2
    node["borderWidthSelected"] = 4
    node["font"] = {"size": 16, "face": "Microsoft YaHei"}
    node["color"] = {"border": "#2B7CE9", "background": "#97C2FC", "highlight": {"border": "#FFA500", "background": "#FFFF00"}}
    
    # 根据关系类型设置不同颜色
    if "叛逃师门" in node.get("note", ""):
        node["color"]["border"] = "#FF0000"
        node["color"]["background"] = "#FF9999"
    elif "已被夺舍" in node.get("note", ""):
        node["color"]["border"] = "#800080"
        node["color"]["background"] = "#CC99FF"

# 设置边样式
for edge in net.edges:
    if edge["title"] == "师徒":
        edge["color"] = "#FF5733"
    elif edge["title"] == "侠缘":
        edge["color"] = "#33FF57"
    elif edge["title"] == "姐妹":
        edge["color"] = "#FF33A8"

# 添加搜索功能
search_box = widgets.Text(
    placeholder='输入名字搜索...',
    description='搜索:',
    layout=widgets.Layout(width='300px')
)

# 创建详细信息显示区域
info_display = widgets.HTML(value="<div style='background:#333; padding:15px; border-radius:10px; min-height:200px; color:#fff; font-family:Microsoft YaHei'>"
                                 "<h3 style='color:#4CAF50'>关系图说明</h3>"
                                 "<p>点击图中任意名字查看详细信息</p >"
                                 "<p>使用搜索框查找特定人物</p >"
                                 "<p>颜色说明：</p >"
                                 "<ul>"
                                 "<li><span style='color:#FF5733'>师徒关系</span></li>"
                                 "<li><span style='color:#33FF57'>侠缘关系</span></li>"
                                 "<li><span style='color:#FF33A8'>姐妹关系</span></li>"
                                 "<li><span style='color:#FF0000'>叛逃师门</span></li>"
                                 "<li><span style='color:#800080'>已被夺舍</span></li>"
                                 "</ul></div>")

# 创建搜索结果显示区域
search_result = widgets.Output()

# 搜索功能处理函数
def handle_search(change):
    with search_result:
        search_result.clear_output()
        search_term = change['new'].strip()
        if not search_term:
            return
        
        # 查找匹配的节点
        matches = [node for node in net.nodes if search_term.lower() in node['label'].lower()]
        
        if not matches:
            print(f"未找到与 '{search_term}' 相关的结果")
            return
        
        # 显示匹配结果
        print(f"找到 {len(matches)} 个匹配结果:")
        for i, node in enumerate(matches, 1):
            print(f"{i}. {node['label']}")
            
        # 高亮显示第一个匹配结果及其关系
        if matches:
            node_id = matches[0]['id']
            net.set_options(f"""
            var options = {{
              nodes: {{
                fixed: {{
                  x: false,
                  y: false
                }}
              }},
              physics: {{
                stabilization: {{
                  enabled: true,
                  iterations: 1000
                }}
              }},
              interaction: {{
                hover: true,
                tooltipDelay: 200
              }}
            }};
            """)
            net.show("network.html")
            display(HTML(filename="network.html"))
            
            # 使用JavaScript高亮节点
            display(HTML(f"""
            <script>
            // 高亮搜索到的节点及其关系
            var nodeId = {node_id};
            var allNodes = nodes.get();
            var connectedNodes = network.getConnectedNodes(nodeId);
            
            // 隐藏所有节点
            allNodes.forEach(function(node) {{
                node.hidden = true;
            }});
            
            // 显示搜索到的节点及其直接关系
            allNodes.forEach(function(node) {{
                if (node.id === nodeId || connectedNodes.includes(node.id)) {{
                    node.hidden = false;
                }}
            }});
            
            // 更新网络
            nodes.update(allNodes);
            network.fit();
            </script>
            """))

search_box.observe(handle_search, names='value')

# 节点点击事件处理
def node_click_callback(node_id):
    node = next(n for n in net.nodes if n['id'] == node_id)
    label = node['label']
    
    # 构建详细信息
    details = f"<div style='padding:15px; background:#333; border-radius:10px; font-family:Microsoft YaHei; color:#fff'>"
    details += f"<h3 style='color:#4CAF50'>{label}</h3>"
    
    # 查找所有关系
    relations_info = []
    for person, data in relations.items():
        # 查找师傅/兄弟姐妹/侠缘
        for rel_type, people in data.items():
            if label in people or label == person:
                # 处理特殊标记
                if label in people:
                    base_label = label.split("（")[0]
                    if base_label == label:  # 没有特殊标记
                        relations_info.append(f"<li><b>{person}</b> 的 {rel_type}</li>")
                    else:
                        relations_info.append(f"<li><b>{person}</b> 的 {rel_type} <span style='color:#FF0000'>({node.get('note', '')})</span></li>")
                else:
                    # 查找徒弟
                    if rel_type == "徒弟":
                        for p in people:
                            base_p = p.split("（")[0]
                            note = p.split("（")[1].replace("）", "") if "（" in p else ""
                            if note:
                                relations_info.append(f"<li><b>{base_p}</b> 的师傅 <span style='color:#FF0000'>({note})</span></li>")
                            else:
                                relations_info.append(f"<li><b>{base_p}</b> 的师傅</li>")
    
    # 添加详细信息
    if relations_info:
        details += "<h4>关系:</h4><ul>"
        for rel in relations_info:
            details += rel
        details += "</ul>"
    else:
        details += "<p>没有找到相关关系信息</p >"
    
    details += "</div>"
    
    # 更新信息显示
    info_display.value = details

# 添加节点点击事件处理
net.on("click", node_click_callback)

# 显示界面
display(HTML("<h1 style='text-align:center; color:#4CAF50; font-family:Microsoft YaHei'>江湖关系图谱</h1>"))
display(widgets.HBox([search_box, widgets.Label(value="按Enter搜索", layout=widgets.Layout(margin='0 0 0 10px'))]))
display(widgets.HTML("<div style='height:20px'></div>"))
display(widgets.HBox([widgets.VBox([info_display, search_result], layout=widgets.Layout(width='30%')), 
                     net.show("network.html"), 
                     layout=widgets.Layout(width='100%')]))
