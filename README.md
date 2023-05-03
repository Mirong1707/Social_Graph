# Social_Graph
This is social networking research 👥. <br/>
Purpose: To find, cluster and classify the strong connectivity components in a graph of friends from a social network using Python and Gephi. <br/>
In order to parse your friends, we've generated an easy-to-analyse json file. The following is one version of parsing VKontakte.
```python
import requests
import json

all_id = []

for i in range(13):
    print(i)
    res = requests.get(f"https://api.vk.com/method/groups.getMembers?group_id=45&offset={i * 1000}&count=1000"
                       f"&access_token=4b32289d4b32289d4b32289d1d4b40471c44b324b32289d15e80a8caa89b73d1d2a5579&v=5.107")
    all_id.extend(json.loads(res.text)['response']['items'])

with open('ids.json', 'w') as f:
    f.write(json.dumps({'ids': all_id}))
```

Next the json was converted into a handy two files. One has a list of all the vertices of the graph, the other describes an edge on each line, with two vertices. Next the code that forms the text file.
```python
txt = "test.txt"
f = open(txt).read()
ans = open('rebra.txt', 'w')
arr = []
flag = False
k = 0
check = 1
for i in f:
    if not (ord('0') <= ord(i) <= ord('9')):
        if k != 0:
            if check == 1:
                ans.write(str(k) + ' ')
            else:
                ans.write(str(k) + '\n')
            check *= -1
        k = 0
        flag = False
    if not flag and ord('0') <= ord(i) <= ord('9'):
        flag = True
    if flag and ord('0') <= ord(i) <= ord('9'):
        k = k * 10 + int(i)
print(arr)

```
Now a table of vertices and edges has been generated in an xlsx file for further analysis. The table generation code will be given below, as it is quite similar when solving the two problems. These tables can now be exported to gephi. This is a graph visualizer. This is what the whole graph looks like. Immediately you can notice a few giant vertices (the size of a graph vertex is proportional to its degree). These are the pages of the two teachers https://vk.com/id238683 and https://vk.com/id133883. They are closely related to the organization of various extracurricular activities in the school, and consequently communicate a lot with all classes in the school, so there are so many different connections. Janina Yadova, for example, gave us the opportunity to get into this internship. 
![image](https://user-images.githubusercontent.com/31445859/172258062-c489ede5-cd30-4a30-ae55-b4eb10cb6f66.png)

Immediately you can see that the nodes with the most connections are usually the teachers, and thus it is not difficult to distinguish them from the general mass. 
Now I have done a little investigation with my pens. As a physical model is applied to the graph, it highlights communities, they look like clustered vertices. I chose a small sub-graph, and characterised each group by contacting some of its members and finding out how they were connected to our school.
![image](https://user-images.githubusercontent.com/31445859/172258080-7ed0d39b-d37e-4a63-89bd-7c8b51cc42f8.png)

A person's class I determined directly by asking everyone on facebook, under the pretext of research. And so, under group 1 in the photo is the community of current grades 11-1 and 11-2. They are so closely related, as they were intermingled over the course of their studies. That is to say, the former when we moved from grade 7 to 8, we were roughly sorted by interests (maths or physics), but the old connections remained. And by and large, the two classes are pretty close, speaking as a student of one of them. Under numbers 2 is 11-3 grade, 3 is 11-5 grade, 4 is 11-7 and in question are 3 people from 11-4. Their class has sparred badly as many of the pages are either unsubscribed to the group or closed. As a result, only a few people are surrounded by other classes from the parallel, and the rest are either smeared amongst other classes or are not in the graph at all. As you can see, the communities are formed quite clearly, and manually separating them is not a problem. 

But it's rather long and impractical to allocate everything by hand. Therefore, an algorithm was invented for selecting a class based on the id of one of its members. That is, the component of strong relatedness to which the member in question belongs is selected. Here is the code that does this.
```python
import requests
import json


def findFriend(id, da):
    ss = set()
    for i in d[id]:
        ss.add(i)
    ds = {}
    for i in ss:
        ds[i] = 0
        for j in d[i]:
            for k in ss:
                if k == j:
                    ds[i] += 1
                    break

    anss = []
    for i in ss:
        anss.append((ds[i], i))
    anss = sorted(anss, reverse=True)
    k = 1
    for i in anss:
        if not da.get(i[1]) is None:
            da[i[1]] += k
            k *= 0.95


def GetNameById(user_id):
    res = requests.get(f"https://api.vk.com/method/users.get?user_ids={user_id}&"
                       f"access_token=4b32289d4b32289d4b32289d1d4b40471c44b324b32289d15e80a8caa89b73d1d2a5579&v=5.107&lang=ru")
    res = json.loads(res.text)['response'][0]
    return res['first_name'] + ' ' + res['last_name']


f = open("rebra.txt").read().split("\n")
d = {}
st = set()
for ind, i in enumerate(f):
    a = 0
    b = 0
    j = 0
    flag = False
    while j < len(i):
        if i[j] == ' ':
            flag = True
            j += 1
            continue
        if not flag:
            a = a * 10 + int(i[j])
        else:
            b = b * 10 + int(i[j])
        j += 1
    st.add(a)
    st.add(b)
    if d.get(a) is None:
        d[a] = []
    if d.get(b) is None:
        d[b] = []
    d[a].append(b)
    d[b].append(a)
id0 = 420403096
suspected = set()
for i in d[id0]:
    suspected.add(i)
ds = {}
dans = {}
for i in suspected:
    dans[i] = 0
ii = 0
findFriend(id0, dans)
ans = []
for i in suspected:
    ans.append((dans[i], i))
ans = sorted(ans, reverse=True)
for i in ans:
    findFriend(i[1], dans)
    if ii > 10:
        break
    ii += 1

ans = []
for i in suspected:
    ans.append((dans[i] / (len(d[i]) + 5), i))
ans = sorted(ans, reverse=True)
jj = 0
for i in ans:
    print(i[0], GetNameById(i[1]), jj)
    jj += 1
    if jj > 30:
        break


```
To describe the algorithm in a nutshell, we simply take all vertices with which the vertex in question is connected, and then distinguish the first 30 members that are closely related to each other. But also this algorithm is applied to several other vertices from the friends of the considered one, in order to exclude false connectivity components, with which the considered vertex is connected by just a couple of edges.
So, I applied this algorithm to my page, and got the following result. It is 90% correct. But also our class teacher (although it may be considered as a class member), and a couple of people from 11-2, as I said before that we have close social ties with them.
![image](https://user-images.githubusercontent.com/31445859/172258100-a2ae4a3e-0c3c-456b-a5cb-fd99a92a015f.png)


Next, a histogram has been made. I now attach the code that generates the xlsx table I referred to above.
```python
from openpyxl import load_workbook

wb = load_workbook('t1.xlsx')
sheet = wb.get_sheet_by_name('test')
f = open("rebra.txt").read().split("\n")
ans = []
d = {}
dd = {}
st = set()
ii = 0
for ind, i in enumerate(f):
    ans.append([])
    a = 0
    b = 0
    j = 0
    flag = False
    while j < len(i):
        if i[j] == ' ':
            flag = True
            j += 1
            continue
        if not flag:
            a = a * 10 + int(i[j])
        else:
            b = b * 10 + int(i[j])
        j += 1
    if(a > b):
        a, b = b, a
    if dd.get((a, b)) is None:
        dd[(a, b)] = 1
        st.add(a)
        st.add(b)
        if d.get(a) is None:
            d[a] = 0
        if d.get(b) is None:
            d[b] = 0
        d[a] += 1
        d[b] += 1
st1 = set()
d1 = {}
check = 0
for i in st:
    st1.add(d[i])
    check += 1
    if d1.get(d[i]) is None:
        d1[d[i]] = 0
    d1[d[i]] += 1
ans1 = []
iii = 0
for i in st1:

    ans1.append([i, d1[i]])
    sheet[f'A{iii + 2}'] = i
    sheet[f'B{iii + 2}'] = d1[i]
    iii += 1
ans1 = sorted(ans1)
print(ans1)

wb.save('t1.xlsx')

```
The constructed table stores two columns. The first has the number of links and the second the number of nodes with that number of links. A graph has been plotted using this data.
![image](https://user-images.githubusercontent.com/31445859/172258118-6ea3dcba-9373-4498-a374-bda3e10d38db.png)

It is hard to speculate why the distribution is the way it is. It is especially strange that there are so many nodes with so few connections. In my opinion it can be explained by the fact that people interested in our school sign up for groups. But the interest rarely develops into the fact that a person or his/her children come here and they forget to unsubscribe from the group. It is worth remembering that the first point is missing on the graph, for 0 connections, where there are about 2500 nodes, which confirms the above theory
In the same way a study of the age dependence of the participants in the group was conducted. The method of forming the table is the same. Here is the resulting graph of dependence. It is clear, that it is not exact, as many specify incorrect age (more than 100 years), but in the average dependence makes sense.
![image](https://user-images.githubusercontent.com/31445859/172258133-60b67dc4-e6ac-4ea5-99d9-7444517c686d.png)

This concludes the study. Thank you for your attention.
