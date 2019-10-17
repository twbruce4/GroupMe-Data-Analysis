
# GroupMe Data Scraping

## To download your GroupMe data, go to your GroupMe profile on the web version.  Click on "Export My Data", then "Create Export".  You can choose which groups and/or direct messages to export.  The data I used for information about likes is in the message.json file.


```python
import pandas as pd

df = pd.read_json('message_original_squad.json')

df['like_num'] = df['favorited_by'].apply(len)
df.head()
name_pivtab = pd.pivot_table(df,index='sender_id',columns='name',aggfunc='count').fillna(0)

def get_max_name(s):
    if s in name_pivtab.T:
        return name_pivtab.T[s].idxmax()[1]
    else:
        return s
df['Name'] = df['sender_id'].apply(get_max_name)

df_likes = df.groupby('Name')['like_num'].agg('sum').sort_values(ascending = False)

df_likes = pd.DataFrame(df_likes)
df_likes
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>like_num</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Eric Warnky</th>
      <td>162</td>
    </tr>
    <tr>
      <th>Thomas B</th>
      <td>146</td>
    </tr>
    <tr>
      <th>Suhas Kasireddy</th>
      <td>79</td>
    </tr>
    <tr>
      <th>Jason Lee</th>
      <td>68</td>
    </tr>
    <tr>
      <th>Kunal Jain</th>
      <td>55</td>
    </tr>
    <tr>
      <th>Ali Mehmood</th>
      <td>30</td>
    </tr>
    <tr>
      <th>Guo Derek</th>
      <td>20</td>
    </tr>
    <tr>
      <th>Davis Berke</th>
      <td>11</td>
    </tr>
    <tr>
      <th>Vijay Sood</th>
      <td>8</td>
    </tr>
    <tr>
      <th>Justin J</th>
      <td>8</td>
    </tr>
    <tr>
      <th>GroupMe</th>
      <td>7</td>
    </tr>
    <tr>
      <th>GroupMe Calendar</th>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



### Now that we have the number of likes received, we can figure out the number of likes given and compare them.


```python
def get_max_name_list(l):
    l_new=list()
    for s in l:
        l_new.append(get_max_name(s))
    return l_new
df['liked_by_names'] = df['favorited_by'].apply(get_max_name_list)
df['liked_by_names']
liked_series = df[df['like_num']>0]
liked_series['liked_by_names']

id_name_dict = {}
for sender_id in df['sender_id'].unique():
    id_name_dict[sender_id] = get_max_name(sender_id)
    

```


```python
df_likes_given = df.groupby('Name')['created_at'].agg('count')
df_likes_given = pd.DataFrame(df_likes_given)

def likes_given(s):
    return sum([1*(s in name) for name in liked_series['liked_by_names']])


df_likes_given['Name'] = df_likes_given.index.values
df_likes_given['likes_given'] = df_likes_given['Name'].apply(likes_given)
df_likes_given = df_likes_given.drop('Name',axis=1)

```


```python
df_likes_proportion = pd.merge(df_likes_given,df_likes,left_on = 'Name',right_on = 'Name')
df_likes_proportion = df_likes_proportion.rename(columns = {'created_at':'messages','like_num':'likes'})
df_likes_proportion['likes_per_messages'] = df_likes_proportion['likes']/df_likes_proportion['messages']
df_likes_proportion['likes_per_messages'].sort_values(ascending=False)
df_likes_proportion['ratio_of_received_to_given'] = df_likes_proportion['likes']/df_likes_proportion['likes_given']
df_likes_proportion = df_likes_proportion.sort_values(by = 'likes', ascending = False)
df_likes_proportion
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>messages</th>
      <th>likes_given</th>
      <th>likes</th>
      <th>likes_per_messages</th>
      <th>ratio_of_received_to_given</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Eric Warnky</th>
      <td>325</td>
      <td>43</td>
      <td>162</td>
      <td>0.498462</td>
      <td>3.767442</td>
    </tr>
    <tr>
      <th>Thomas B</th>
      <td>572</td>
      <td>188</td>
      <td>146</td>
      <td>0.255245</td>
      <td>0.776596</td>
    </tr>
    <tr>
      <th>Suhas Kasireddy</th>
      <td>405</td>
      <td>180</td>
      <td>79</td>
      <td>0.195062</td>
      <td>0.438889</td>
    </tr>
    <tr>
      <th>Jason Lee</th>
      <td>299</td>
      <td>55</td>
      <td>68</td>
      <td>0.227425</td>
      <td>1.236364</td>
    </tr>
    <tr>
      <th>Kunal Jain</th>
      <td>279</td>
      <td>12</td>
      <td>55</td>
      <td>0.197133</td>
      <td>4.583333</td>
    </tr>
    <tr>
      <th>Ali Mehmood</th>
      <td>43</td>
      <td>65</td>
      <td>30</td>
      <td>0.697674</td>
      <td>0.461538</td>
    </tr>
    <tr>
      <th>Guo Derek</th>
      <td>34</td>
      <td>8</td>
      <td>20</td>
      <td>0.588235</td>
      <td>2.500000</td>
    </tr>
    <tr>
      <th>Davis Berke</th>
      <td>69</td>
      <td>27</td>
      <td>11</td>
      <td>0.159420</td>
      <td>0.407407</td>
    </tr>
    <tr>
      <th>Justin J</th>
      <td>28</td>
      <td>3</td>
      <td>8</td>
      <td>0.285714</td>
      <td>2.666667</td>
    </tr>
    <tr>
      <th>Vijay Sood</th>
      <td>13</td>
      <td>13</td>
      <td>8</td>
      <td>0.615385</td>
      <td>0.615385</td>
    </tr>
    <tr>
      <th>GroupMe</th>
      <td>26</td>
      <td>0</td>
      <td>7</td>
      <td>0.269231</td>
      <td>inf</td>
    </tr>
    <tr>
      <th>GroupMe Calendar</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



### A quick scatterplot shows that the number of likes given and likes received tends to be positively correlated, indicating that group members that are active in the chat both like messages and have their messages liked.


```python
import matplotlib.pyplot as plt
plt.scatter(df_likes_proportion['likes'],df_likes_proportion['likes_given'])
plt.xlabel('Likes Received')
plt.ylabel('Likes Given')
plt.show()
```


![png](groupme_thomas_files/groupme_thomas_7_0.png)


### This code creates a "biggest fan" attribute, which determines who has liked the most of each person's messages.


```python
import numpy as np
liked_df_group = liked_series.groupby('Name')

def get_likes_in_group(s, name):
    return sum([1*(name in group) for group in np.array(liked_df_group.get_group(s)['liked_by_names'])])
def get_favorites(s):
    if s in list(liked_series['Name']):
        liked_by_array = np.array(liked_df_group.get_group(s)['liked_by_names'])
        names_list = list()
        num_messages_liked = list()
        for name in df_likes_proportion.index.values:
            names_list.append(name)
            num_messages_liked.append(get_likes_in_group(s,name))
        index = pd.Series(num_messages_liked).idxmax()
        return (names_list[index], int(num_messages_liked[index]))
    else:
        return ''
df_likes_proportion['Name'] = df_likes_proportion.index.values
df_likes_proportion['biggest_fan'] = df_likes_proportion['Name'].apply(get_favorites)
df_likes_proportion = df_likes_proportion.drop('Name',axis=1)
df_likes_proportion

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>messages</th>
      <th>likes_given</th>
      <th>likes</th>
      <th>likes_per_messages</th>
      <th>ratio_of_received_to_given</th>
      <th>biggest_fan</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Eric Warnky</th>
      <td>325</td>
      <td>43</td>
      <td>162</td>
      <td>0.498462</td>
      <td>3.767442</td>
      <td>(Thomas B, 72)</td>
    </tr>
    <tr>
      <th>Thomas B</th>
      <td>572</td>
      <td>188</td>
      <td>146</td>
      <td>0.255245</td>
      <td>0.776596</td>
      <td>(Suhas Kasireddy, 79)</td>
    </tr>
    <tr>
      <th>Suhas Kasireddy</th>
      <td>405</td>
      <td>180</td>
      <td>79</td>
      <td>0.195062</td>
      <td>0.438889</td>
      <td>(Thomas B, 36)</td>
    </tr>
    <tr>
      <th>Jason Lee</th>
      <td>299</td>
      <td>55</td>
      <td>68</td>
      <td>0.227425</td>
      <td>1.236364</td>
      <td>(Thomas B, 31)</td>
    </tr>
    <tr>
      <th>Kunal Jain</th>
      <td>279</td>
      <td>12</td>
      <td>55</td>
      <td>0.197133</td>
      <td>4.583333</td>
      <td>(Thomas B, 21)</td>
    </tr>
    <tr>
      <th>Ali Mehmood</th>
      <td>43</td>
      <td>65</td>
      <td>30</td>
      <td>0.697674</td>
      <td>0.461538</td>
      <td>(Thomas B, 10)</td>
    </tr>
    <tr>
      <th>Guo Derek</th>
      <td>34</td>
      <td>8</td>
      <td>20</td>
      <td>0.588235</td>
      <td>2.500000</td>
      <td>(Thomas B, 7)</td>
    </tr>
    <tr>
      <th>Davis Berke</th>
      <td>69</td>
      <td>27</td>
      <td>11</td>
      <td>0.159420</td>
      <td>0.407407</td>
      <td>(Thomas B, 5)</td>
    </tr>
    <tr>
      <th>Justin J</th>
      <td>28</td>
      <td>3</td>
      <td>8</td>
      <td>0.285714</td>
      <td>2.666667</td>
      <td>(Thomas B, 3)</td>
    </tr>
    <tr>
      <th>Vijay Sood</th>
      <td>13</td>
      <td>13</td>
      <td>8</td>
      <td>0.615385</td>
      <td>0.615385</td>
      <td>(Suhas Kasireddy, 4)</td>
    </tr>
    <tr>
      <th>GroupMe</th>
      <td>26</td>
      <td>0</td>
      <td>7</td>
      <td>0.269231</td>
      <td>inf</td>
      <td>(Suhas Kasireddy, 3)</td>
    </tr>
    <tr>
      <th>GroupMe Calendar</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>


