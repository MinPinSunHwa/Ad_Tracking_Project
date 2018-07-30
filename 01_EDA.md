##### TalkingData AdTracking Fraud Detection Challenge
# 1. EDA
[source code](01_EDA.py)
### Explor data
* train data shape : (184903890, 8)
* test data shape : (18790469, 7)

<br>

* train data columns : <br>
ip, app, device, os, channel, click_time, attributed_time, is_attributed <br>

* test data columns : <br>
click_id, ip, app, device, os, channel, click_time <br>

<br>

* train data info. : <br>


* test data info. : <br>

<br>

### Check download frequency
* 0 : 18,447,044 <br>
* 1 : 456,846 <br>

<br>

### Check 'click_time'
* year count of train data : <br>
2017 : 184,903,890 <br>

<br>

* month count of train data : <br>
11 : 184,903,890 <br>

<br>

* day count of train data : <br>
6 : 9,308,568 <br>
7 : 59,633,310 <br>
8 : 62,945,075 <br>
9 : 53,016,937 <br>

<br>

* year count of test data : <br>
2017 : 18,790,469 <br>

<br>

* momth count of test data : <br>
11 : 18,790,469 <br>

<br>

* day count of test data : <br>
10 : 18,790,469 <br>

<br>

### Draw time series of train data click time
```python
temp = train['click_time']
temp.index = train['click_time']
temp = temp.resample('10T').count()

plt.figure(figsize=(10,5))
plt.title('click time (10 minute bins) of train data')
plt.plot(temp.index, temp, 'g')
plt.xticks(label=[])
plt.savefig('graph/train_click_time.png')
plt.show()
gc.collect()
```

![png](graph/train_click_time.png)

<br>

### Draw time series of test data click time
```python
temp = test['click_time']
temp.index = test['click_time']
temp = temp.resample('10T').count()

plt.figure(figsize=(10,5))
plt.title('click time (10 minute bins) of test data')
plt.plot(temp.index, temp, 'g')
plt.xticks(rotation=30, fontsize="small")
plt.savefig('graph/test_click_time.png')
plt.show()
gc.collect()
```

![png](graph/test_click_time.png)

<br>

### Draw time series of click time and attributed time
```python
temp1 = train['is_attributed']
temp1.index = train['click_time']
temp1 = temp1.resample('10T').sum()

temp2 = train['is_attributed']
temp2.index = train['attributed_time']
temp2 = temp2.resample('10T').sum()

plt.figure(figsize=(10,5))
plt.title('click time and attributed time')
plt.plot(temp1.index, temp1, 'g', label='click time')
plt.plot(temp2.index, temp2, 'r', label='attributed time')
plt.legend(loc='lower right', fontsize='small')
plt.savefig('graph/train_click_download.png')
plt.show()
gc.collect()
```

![png](graph/train_click_download.png)

<br>

### Make a derived variable : hour
```python
train['hour'] = np.nan
train['hour'] = train['click_time'].dt.hour

test['hour'] = np.nan
test['hour'] = test['click_time'].dt.hour
gc.collect()
```

<br>

### Draw hour bar graphs
```python
plt.figure(figsize=(15,10))

plt.subplot(2,1,1)
plt.title('click count per hour in train data')
sns.countplot('hour', data=train, linewidth=0)

plt.subplot(2,1,2)
plt.title('click count per hour in test data')
sns.countplot('hour', data=test, linewidth=0)

plt.savefig('graph/hour_click_count.png')
plt.show()
gc.collect()
```

![png](graph/hour_click_count.png)

<br>

### Draw hour and download bar graphs
```python
plt.figure(figsize=(15,15))

plt.subplot(3,1,1)
plt.title('click count per hour in train data')
sns.countplot('hour', data=train)

plt.subplot(3,1,2)
plt.title('download count per hour')
sns.barplot('hour', 'is_attributed', data=train, estimator=sum, ci=None)

plt.subplot(3,1,3)
plt.title('download rate by per hour')
sns.barplot('hour', 'is_attributed', data=train, ci=None)

plt.savefig('graph/hour_download_rate.png')
plt.show()
gc.collect()
```

![png](graph/hour_download_rate.png)

<br>

### Merge trian data and test data
```python
del train['attributed_time']
test['is_attributed'] = 0
data = pd.concat([train, test])

del train
del test
gc.collect()
```

merged data shape : (203694359, 9)

<br>

### Make black list
```python
def make_black_list(v):
    temp = data[v].value_counts().reset_index()
    temp.columns = [v,'count']
    temp.sort_values(ascending=False, by='count', inplace=True)

    temp2 = data.groupby(v)['is_attributed'].sum().reset_index()
    temp2.columns = [v,'download']
    temp = temp.merge(temp2, on=v, how='left')

    print('sort by count')
    print(temp.head(30))
    print(temp.tail(30))
    print()

    temp.sort_values(ascending=False, by='download', inplace=True)

    print('sort by download')
    print(temp.head(30))
    print(temp.tail(30))
    print()

    temp['gap'] = temp['count'] - temp['download']
    temp.sort_values(ascending=False, by='gap', inplace=True)

    print('sort by gap')
    print(temp.head(30))
    print(temp.tail(30))
    print()

    temp['rate'] = temp['download'] / temp['count']
    temp.sort_values(ascending=False, by='rate', inplace=True)

    print('sort by rate')
    print(temp.head(30))
    print(temp.tail(30))
    print()

    black_boundary = temp['count'].median() + 10
    print('black list boundary : ', black_boundary)

    temp['black_' + v] = 0
    temp.loc[(temp['count'] > black_boundary) & (temp['rate'] == 0), 'black_' + v] = 1
    temp.sort_values(by=v, inplace=True)

    print('check black list')
    print(temp.head(30))
    print(temp.tail(30))
    print('count : ', temp['black_' + v].sum())

    return temp

ip = make_black_list('ip')
app = make_black_list('app')
device = make_black_list('device')
os = make_black_list('os')
channel = make_black_list('channel')
hour = make_black_list('hour')
```

* ip black list : 34,770 <br>
* app black list : 178 <br>
* device black list : 62 <br>
* os black list : 170 <br>
* channel black list : 0 <br>
* hour black list : 0 <br>

<br>

### Draw bar graphs
```python
def barplot(data, v):
    temp1 = data.head(30)
    temp2 = data.tail(30)
    plt.figure(figsize=(20,15))

    plt.subplot(3,2,1)
    plt.title('click count per ' + v + ' (top)')
    sns.barplot(v, 'count', data=temp1, linewidth=0)
    plt.xticks(rotation=90, fontsize="small")

    plt.subplot(3,2,2)
    plt.title('(bottom)')
    sns.barplot(v, 'count', data=temp2, linewidth=0)
    plt.xticks(rotation=90, fontsize="small")

    plt.subplot(3,2,3)
    plt.title('download count per ' + v + ' (top)')
    sns.barplot(v, 'download', data=temp1, linewidth=0, estimator=sum, ci=None)
    plt.xticks(rotation=90, fontsize="small")

    plt.subplot(3,2,4)
    plt.title('(bottom)')
    sns.barplot(v, 'download', data=temp2, linewidth=0, estimator=sum, ci=None)
    plt.xticks(rotation=90, fontsize="small")

    plt.subplot(3,2,5)
    plt.title('download rate per ' + v + ' (top)')
    sns.barplot(v, 'rate', data=temp1, linewidth=0, ci=None)
    plt.xticks(rotation=90, fontsize="small")

    plt.subplot(3,2,6)
    plt.title('(bottom)')
    sns.barplot(v, 'rate', data=temp2, linewidth=0, ci=None)
    plt.xticks(rotation=90, fontsize="small")

    plt.savefig('graph/' + v + '_download_rate.png')
    plt.show()
    gc.collect()
```

```python
ip.sort_values(ascending=False, by='count', inplace=True)
barplot(ip, 'ip')
```

![png](graph/ip_download_rate.png)

```python
app.sort_values(ascending=False, by='count', inplace=True)
barplot(app, 'app')
```

![png](graph/app_download_rate.png)

```python
device.sort_values(ascending=False, by='count', inplace=True)
barplot(device, 'device')
```

![png](graph/device_download_rate.png)

```python
os.sort_values(ascending=False, by='count', inplace=True)
barplot(os, 'os')
```

![png](graph/os_download_rate.png)

```python
channel.sort_values(ascending=False, by='count', inplace=True)
barplot(channel, 'channel')
```

![png](graph/channel_download_rate.png)

<br>

### Draw scatter plots
```python
def scatter_plot(feat, file_name):
    temp = data[feat].loc[data['click_id'].isnull()]

    g = sns.pairplot(temp,
                     hue='is_attributed',
                     palette="husl",
                     plot_kws={'alpha':0.1})

    for ax in g.axes.flat:
        for label in ax.get_xticklabels():
            label.set_rotation(90)

    g.fig.set_size_inches(10,10)
    plt.savefig('graph/'+file_name+'.png')
    plt.show()
    gc.collect()
```

```python
feat = ['is_attributed', 'ip']
scatter_plot(feat, 'scatter_plot_ip')
```

![png](graph/scatter_plot_ip.png)

```python
feat = ['is_attributed', 'app']
scatter_plot(feat, 'scatter_plot_app')
```

![png](graph/scatter_plot_app.png)

```python
feat = ['is_attributed', 'device']
scatter_plot(feat, 'scatter_plot_device')
```

![png](graph/scatter_plot_device.png)

```python
feat = ['is_attributed', 'os']
scatter_plot(feat, 'scatter_plot_os')
```

![png](graph/scatter_plot_os.png)

```python
feat = ['is_attributed', 'channel']
scatter_plot(feat, 'scatter_plot_channel')
```

![png](graph/scatter_plot_channel.png)

```python
feat = ['is_attributed', 'hour']
scatter_plot(feat, 'scatter_plot_hour')
```

![png](graph/scatter_plot_hour.png)