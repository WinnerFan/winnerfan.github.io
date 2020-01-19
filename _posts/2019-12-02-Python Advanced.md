### Python

#### matplotlib

```
import matplotlib.pyplot as plt

# default x is from 0
input_values = [1,2,3,4,5]
squares = [1,4,9,16,25]
#plt.plot(squares, lineWidth=5)
plt.plot(input_values, squares, lineWidth=5)
plt.title("Square Numbers", fontsize=24)
plt.xlabel("Value", fontsize=14)
plt.ylabel("Square of Value", fontsize=14)
plt.tick_params(axis='both', labelsize=14)
plt.axis([0,6,0,36])
plt.axes().get_xaxis().set_visible(False)
plt.axes().get_yaxis().set_visible(False)
plt.figure(dpi=128, figsize=(10,6))

plt.scatter(input_values, squares,c=(0,0,0.8), edgecolor='none', s=200)
#plt.scatter(input_values, squares,c=input_values, cmap=plt.cm.Blues, edgecolor='none', s=200)

plt.show()
plt.savefig('squares_plot.png', bbox_inches='tight')
```

#### csv

```
import csv

filename = 'sitka_weather_07-2014.csv'
with open(filename) as f:
	reader = csv.reader(f)
	# next和for会修改迭代器起点
	a = [i for i in reader]
	print(a[1][1])

	#head_row = next(reader)
	#for index, col in enumerate(head_row):
	#	print(index, col)

```

#### json

```
import json

filename = 'test.json'
with open(filename) as f:
	data = json.load(f)

family = data[0]['BaseSettings']['size']   
```