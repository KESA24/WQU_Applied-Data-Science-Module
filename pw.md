```python
%logstop
%logstart -rtq ~/.logs/pw.py append
import seaborn as sns
sns.set()
```


```python
from static_grader import grader
```

# PW Miniproject
## Introduction

The objective of this miniproject is to exercise your ability to use basic Python data structures, define functions, and control program flow. We will be using these concepts to perform some fundamental data wrangling tasks such as joining data sets together, splitting data into groups, and aggregating data into summary statistics.
**Please do not use `pandas` or `numpy` to answer these questions.**

We will be working with medical data from the British NHS on prescription drugs. Since this is real data, it contains many ambiguities that we will need to confront in our analysis. This is commonplace in data science, and is one of the lessons you will learn in this miniproject.

## Downloading the data

We first need to download the data we'll be using from Amazon S3:


```bash
%%bash
mkdir pw-data
wget http://dataincubator-wqu.s3.amazonaws.com/pwdata/201701scripts_sample.json.gz -nc -P ./pw-data
wget http://dataincubator-wqu.s3.amazonaws.com/pwdata/practices.json.gz -nc -P ./pw-data
```

    mkdir: cannot create directory ‘pw-data’: File exists
    File ‘./pw-data/201701scripts_sample.json.gz’ already there; not retrieving.
    
    File ‘./pw-data/practices.json.gz’ already there; not retrieving.
    


## Loading the data

The first step of the project is to read in the data. We will discuss reading and writing various kinds of files later in the course, but the code below should get you started.


```python
import gzip
import simplejson as json
```


```python
with gzip.open('./pw-data/201701scripts_sample.json.gz', 'rb') as f:
    scripts = json.load(f)

with gzip.open('./pw-data/practices.json.gz', 'rb') as f:
    practices = json.load(f)
```

This data set comes from Britain's National Health Service. The `scripts` variable is a list of prescriptions issued by NHS doctors. Each prescription is represented by a dictionary with various data fields: `'practice'`, `'bnf_code'`, `'bnf_name'`, `'quantity'`, `'items'`, `'nic'`, and `'act_cost'`. 


```python
scripts[:2]
```




    [{'bnf_code': '0101010G0AAABAB',
      'items': 2,
      'practice': 'N81013',
      'bnf_name': 'Co-Magaldrox_Susp 195mg/220mg/5ml S/F',
      'nic': 5.98,
      'act_cost': 5.56,
      'quantity': 1000},
     {'bnf_code': '0101021B0AAAHAH',
      'items': 1,
      'practice': 'N81013',
      'bnf_name': 'Alginate_Raft-Forming Oral Susp S/F',
      'nic': 1.95,
      'act_cost': 1.82,
      'quantity': 500}]



A [glossary of terms](http://webarchive.nationalarchives.gov.uk/20180328130852tf_/http://content.digital.nhs.uk/media/10686/Download-glossary-of-terms-for-GP-prescribing---presentation-level/pdf/PLP_Presentation_Level_Glossary_April_2015.pdf/) and [FAQ](http://webarchive.nationalarchives.gov.uk/20180328130852tf_/http://content.digital.nhs.uk/media/10048/FAQs-Practice-Level-Prescribingpdf/pdf/PLP_FAQs_April_2015.pdf/) is available from the NHS regarding the data. Below we supply a data dictionary briefly describing what these fields mean.

| Data field |Description|
|:----------:|-----------|
|`'practice'`|Code designating the medical practice issuing the prescription|
|`'bnf_code'`|British National Formulary drug code|
|`'bnf_name'`|British National Formulary drug name|
|`'quantity'`|Number of capsules/quantity of liquid/grams of powder prescribed|
| `'items'`  |Number of refills (e.g. if `'quantity'` is 30 capsules, 3 `'items'` means 3 bottles of 30 capsules)|
|  `'nic'`   |Net ingredient cost|
|`'act_cost'`|Total cost including containers, fees, and discounts|

The `practices` variable is a list of member medical practices of the NHS. Each practice is represented by a dictionary containing identifying information for the medical practice. Most of the data fields are self-explanatory. Notice the values in the `'code'` field of `practices` match the values in the `'practice'` field of `scripts`.


```python
practices[:2]
```




    [{'code': 'A81001',
      'name': 'THE DENSHAM SURGERY',
      'addr_1': 'THE HEALTH CENTRE',
      'addr_2': 'LAWSON STREET',
      'borough': 'STOCKTON ON TEES',
      'village': 'CLEVELAND',
      'post_code': 'TS18 1HU'},
     {'code': 'A81002',
      'name': 'QUEENS PARK MEDICAL CENTRE',
      'addr_1': 'QUEENS PARK MEDICAL CTR',
      'addr_2': 'FARRER STREET',
      'borough': 'STOCKTON ON TEES',
      'village': 'CLEVELAND',
      'post_code': 'TS18 2AW'}]



In the following questions we will ask you to explore this data set. You may need to combine pieces of the data set together in order to answer some questions. Not every element of the data set will be used in answering the questions.

## Question 1: summary_statistics

Our beneficiary data (`scripts`) contains quantitative data on the number of items dispensed (`'items'`), the total quantity of item dispensed (`'quantity'`), the net cost of the ingredients (`'nic'`), and the actual cost to the patient (`'act_cost'`). Whenever working with a new data set, it can be useful to calculate summary statistics to develop a feeling for the volume and character of the data. This makes it easier to spot trends and significant features during further stages of analysis.

Calculate the sum, mean, standard deviation, and quartile statistics for each of these quantities. Format your results for each quantity as a list: `[sum, mean, standard deviation, 1st quartile, median, 3rd quartile]`. We'll create a `tuple` with these lists for each quantity as a final result.


```python
from math import ceil, floor

def quantile(values, q):
    idx = len(values) * q
    if idx == int(idx):
        return sorted(values)[idx]
    else:
        f1 = floor(idx)
        c1 = ceil(idx)
        return sum(sorted(values)[f1:c1+1]) / 2

def mean(values):
    return sum(values) / len(values)
```


```python
from math import ceil
def std(values, avg):
    numerator = sum([(x-avg)**2 for x in values])
    denominator = len(values)
    return (numerator/denominator)**0.5

def quantile(values, q):
    idx = int(ceil(len(values) * q))
    return sorted(values)[idx]
    
def describe(key):
    values = [d[key] for d in scripts]
    total  = sum(values)
    avg    = mean(values)
    s      = std(values, avg)
    q25 = quantile(values, 0.25)
    med = quantile(values, 0.5)
    q75 =  quantile(values, 0.75)

    return (total, avg, s, q25, med, q75)
```


```python
summary = [('items', describe( 'items')),
           ('quantity', describe( 'quantity')),
           ('nic', describe( 'nic')),
           ('act_cost', describe('act_cost'))]
print(summary)
```

    [('items', (4410054, 11.522744731217633, 33.11216633980368, 1, 3, 8)), ('quantity', (316356836, 826.5883059943667, 3872.1810146096263, 30, 120, 466)), ('nic', (29048309.790000338, 75.89844899484315, 197.5728266277507, 7.7, 22.62, 65.95)), ('act_cost', (27053937.599999707, 70.68748295124895, 183.26731895303854, 7.25, 21.24, 61.54))]



```python
grader.score.pw__summary_statistics(summary)
```

    ==================
    Your score: 1.000
    ==================


## Question 2: most_common_item

Often we are not interested only in how the data is distributed in our entire data set, but within particular groups -- for example, how many items of each drug (i.e. `'bnf_name'`) were prescribed? Calculate the total items prescribed for each `'bnf_name'`. What is the most commonly prescribed `'bnf_name'` in our data?

To calculate this, we first need to split our data set into groups corresponding with the different values of `'bnf_name'`. Then we can sum the number of items dispensed within in each group. Finally we can find the largest sum.

We'll use `'bnf_name'` to construct our groups. You should have *5619* unique values for `'bnf_name'`.


```python
bnf_names = {script['bnf_name'] for script in scripts}
assert(len(bnf_names) == 5619)
```

We want to construct "groups" identified by `'bnf_name'`, where each group is a collection of prescriptions (i.e. dictionaries from `scripts`). We'll construct a dictionary called `groups`, using `bnf_names` as the keys. We'll represent a group with a `list`, since we can easily append new members to the group. To split our `scripts` into groups by `'bnf_name'`, we should iterate over `scripts`, appending prescription dictionaries to each group as we encounter them.


```python
groups = {name: [] for name in bnf_names}
for script in scripts:
    bnf_name = script['bnf_name']
    groups[bnf_name].append(script)
```

Now that we've constructed our groups we should sum up `'items'` in each group and find the `'bnf_name'` with the largest sum. The result, `max_item`, should have the form `[(bnf_name, item total)]`, e.g. `[('Foobar', 2000)]`.


```python
def total_prescriptions(data,key):
    return sum([d[key] for d in data])

total_items_by_bnf_name = []
for bnf_name, scripts_ in groups.items():
    total = total_prescriptions(scripts_, 'items')
    total_items_by_bnf_name.append((bnf_name, total))
max_item = [max(total_items_by_bnf_name, key=lambda x: x[1])]
```

**TIP:** If you are getting an error from the grader below, please make sure your answer conforms to the correct format of `[(bnf_name, item total)]`.


```python
grader.score.pw__most_common_item(max_item)
```

    ==================
    Your score: 1.000
    ==================


**Challenge:** Write a function that constructs groups as we did above. The function should accept a list of dictionaries (e.g. `scripts` or `practices`) and a tuple of fields to `groupby` (e.g. `('bnf_name',)` or `('bnf_name', 'post_code')`) and returns a dictionary of groups. The following questions will require you to aggregate data in groups, so this could be a useful function for the rest of the miniproject.


```python
#Get unique names
bnf_names = {d['bnf_name'] for d in scripts}

#Use unique names to create group dict
groups = {name: [] for name in bnf_names}

#Fill group dict from items in JSON(scripts)
for script in scripts:
    groups[script['bnf_name']].append(script)
```


```python
def group_by_field(data, fields):
    #Get unique names
    names = {tuple(d[field] for field in fields) for d in data}

    #Use unique names to create group dict
    groups = {name: [] for name in names}

    #Fill group dict from items in JSON(scripts)
    for item in data:
        key = tuple(item[field] for field in fields)
        groups[key].append(item)
    
    return groups
```


```python
groups = group_by_field(scripts, ('bnf_name',))

items = [(name, sum([d['items'] for d in group]))
         for name, group in groups.items()]

test_max_item = max(items, key=lambda tup: tup[1])

test_max_item = [(test_max_item[0][0], test_max_item[1])]

assert test_max_item == max_item
```

## Question 3: postal_totals

Our data set is broken up among different files. This is typical for tabular data to reduce redundancy. Each table typically contains data about a particular type of event, processes, or physical object. Data on prescriptions and medical practices are in separate files in our case. If we want to find the total items prescribed in each postal code, we will have to _join_ our prescription data (`scripts`) to our clinic data (`practices`).

Find the total items prescribed in each postal code, representing the results as a list of tuples `(post code, total items prescribed)`. Sort your results ascending alphabetically by post code and take only results from the first 100 post codes. Only include post codes if there is at least one prescription from a practice in that post code.

**NOTE:** Some practices have multiple postal codes associated with them. Use the alphabetically first postal code.

We can join `scripts` and `practices` based on the fact that `'practice'` in `scripts` matches `'code'` in `practices`. However, we must first deal with the repeated values of `'code'` in `practices`. We want the alphabetically first postal codes.


```python
practice_postal = {}
for practice in practices:
    if practice['code'] in practice_postal:
        practice_postal[practice['code']] = practice['post_code'] if practice['post_code'] < practice_postal[practice['code']] else practice_postal[practice['code']]
    else:
        practice_postal[practice['code']] = practice['post_code']
```

**Challenge:** This is an aggregation of the practice data grouped by practice codes. Write an alternative implementation of the above cell using the `group_by_field` function you defined previously.


```python
assert practice_postal['K82019'] == 'HP21 8TR'
```

Now we can join `practice_postal` to `scripts`.


```python
joined = scripts[:]
for script in joined:
    script['post_code'] = practice_postal[script['practice']]
```

Finally we'll group the prescription dictionaries in `joined` by `'post_code'` and sum up the items prescribed in each group, as we did in the previous question.


```python
items_by_post = {}
for join in joined:
    if join['post_code'] in items_by_post:
        items_by_post[join['post_code']]=items_by_post[join['post_code']]+join['items']
    else:
        items_by_post[join['post_code']]=join['items']
```


```python
sort_ = sorted(items_by_post)
postal_total = []
for key in sort_:
    postal_total.append((key, items_by_post[key]))

postal_totals = postal_total[:100]

grader.score.pw__postal_totals(postal_totals)
```

    ==================
    Your score: 1.000
    ==================


## Question 4: items_by_region

Now we'll combine the techniques we've developed to answer a more complex question. Find the most commonly dispensed item in each postal code, representing the results as a list of tuples (`post_code`, `bnf_name`, amount dispensed as proportion of total). Sort your results ascending alphabetically by post code and take only results from the first 100 post codes.

**NOTE:** We'll continue to use the `joined` variable we created before, where we've chosen the alphabetically first postal code for each practice. Additionally, some postal codes will have multiple `'bnf_name'` with the same number of items prescribed for the maximum. In this case, we'll take the alphabetically first `'bnf_name'`.

There are several approaches to solve this problem but we will guide you through one of them. Feel free to solve it your own way if it is easier for you to understand and implement. If your kernel keeps on dying, it's probably an indication that you are running out of memory. Consider deleting objects you don't need anymore using the `del` statement and shutdown any other running notebooks. For example:
```Python
del some_object_not_needed
```

The first step is to calculate the total items for each `'post_code'` and `'bnf_name'`. Let's call that result `total_items_by_post_bnf`. Consider what is the best data structure(s) to represent `total_items_by_post_bnf`. It should have 141196 `('post_code', 'bnf_name')` groups.


```python
def group_by_field(data,fields):
    names={tuple(dict_[field] for field in fields)
           for dict_ in data}
    groups= {name: [] for name in names}
    for dict_ in data:
        name = tuple(dict_[field] for field in fields)
        groups[name].append(dict_)
    return groups
```

Next, let's take `total_items_by_post_bnf` and group it by `'post_code'`. In other words, we want a  data structure that maps a `'post_code'` to a list of all records that belong to that `'post_code'`. There should be 118 groups.


```python
total_items_by_bnf_post =  {}
for key, group in list(group_by_field(joined, ('post_code', 'bnf_name')).items()):
    items_total=sum(d['items'] for d in group)
    total_items_by_bnf_post[key]=items_total
assert len(total_items_by_bnf_post) == 141196
```

Now with `grouped_post_code`, let's iterate over each group and calculate the following fields for each `'post_code'`:
1. the sum of total items for all `'bnf_name'`
1. the most total items
1. the `'bnf_name'` that had the most total items

Once again, consider the best data structure(s) to use to represent the result. It may help to write and use a function when developing your solution.


```python
total_items = []
for (post_code, bnf_name), total in list(total_items_by_bnf_post.items()):
        new_dict = {'post_code' : post_code,
                    'bnf_name' : bnf_name,
                    'total' : total}
        total_items.append(new_dict)
```

Now, we are ready to:
1. calculate the ratio (the amount dispensed as proportion of total)
1. [sort](https://docs.python.org/3/howto/sorting.html) alphabetically by the post code
1. format the answer as a list of tuples
1. take only the first 100 tuples
1. submit to the grader


```python
max_item_by_post = []
total_items_by_post=group_by_field(total_items, ('post_code',))
assert len(total_items_by_post) == 118
```


```python
max_item_by_post = max(total_items_by_post)
from operator import itemgetter
get_total = itemgetter('total')
max_item_by_post = []
groups = list(total_items_by_post.values())
for group in groups:
    max_total = sorted(group, key=itemgetter('total'), reverse=True)[0]
    max_item_by_post.append(max_total)
```


```python
max_item_by_post = [sorted(group, key=itemgetter('total'), reverse=True)[0]
                    for group in list(total_items_by_post.values())]
```


```python
items_by_region=[]
for item in max_item_by_post:
    numerator= item['total']
    denominator=dict(items_by_post)[item['post_code']]
    proportion=numerator/denominator
    result=(item['post_code'], item['bnf_name'], proportion)
    items_by_region.append(result)
items_by_region = sorted(items_by_region)[:100]
```


```python
grader.score.pw__items_by_region(items_by_region)
```

    ==================
    Your score: 1.000
    ==================


*Copyright &copy; 2021 WorldQuant University. This content is licensed solely for personal use. Redistribution or publication of this material is strictly prohibited.*
