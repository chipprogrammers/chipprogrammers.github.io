---
layout: post
title: PCPL RFID Project
category: Electronics
color: teal
theme_color: "#009688"
img: /static/IMG/raspberry_pi.jpg
prism: true
tags: 
- Raspberry Pi
- Python
---

Clone my github repository by executing following commands one by one:
<pre class="line-numbers">
<code class="language-bash">
{% raw %}
cd ~
git clone https://github.com/arunksoman/PCPL-electro.git
cd PCPL-electro
{% endraw %}
</code>
</pre>

Following commands are necessary for install Python library used to read RC522 in current working directory:
<pre class="line-numbers">
<code class="language-bash">
{% raw %}
cd my-project
python setup.py install
cd ..
{% endraw %}
</code>
</pre>

## Run Program
- Rig up the circuit as discussed in class.
- Run the program using main.py using Thonny Python IDE or using **python3 main.py** from terminal .
- Enter Budget from keypad (In order to verify working of program you have to enter budget between 10,000 and 12,000.
- Read RFID tags (white flat tags) one by one from  1 to 5.

### RC 522 Connections

| PINS | Raspberry pi Physical Pin     |
| ----------|:------------------------:|
| SDA | Pin 24 |
| SCK | Pin 23 |
| MOSI | Pin 19 |
| MISO | pin 21 |
| GND | Pin 6 |
| RST | pin 22 |
| 3.3v | pin 1|

### Keypad Connections

- rows = [8, 37, 11, 12]
- cols = [32, 33, 35,36]

## Explanation: main.py

Import necessary modules and packages:
<pre class="line-numbers">
<code class="language-python">
{% raw %}
import RPi.GPIO as GPIO
import time
import sys
from mfrc522 import SimpleMFRC522
{% endraw %}
</code>
</pre>

Choosing Physical pin numbering as BOARD and warnings about using GPIO channels are neglected using GPIO.setwarnings.

<pre class="line-numbers" data-start="5">
<code class="language-python">
{% raw %}
GPIO.setmode(GPIO.BOARD)
GPIO.setwarnings(False)
{% endraw %}
</code>
</pre>

Rows of the keypad is connected to GPIO physical pins  8, 37, 11, and 12. While columns are connected to GPIO physical pins ( as per pin BOARD number) 32, 33, 35, and 36.
<pre class="line-numbers" data-start="7">
<code class="language-python">
{% raw %}
rows = [8, 37, 11, 12]
cols = [32, 33, 35,36]
keys = [
    ['1', '2', '3', 'A'],
    ['4', '5', '6', 'B'],
    ['7', '8', '9', 'C'],
    ['*', '0', '#', 'D']]
{% endraw %}
</code>
</pre>

{% include MyNote.html note_type="warning" span_note="Warning: " text="By pressing keys we are actually putting a short circuit between two GPIO pins. So that you have to make sure that columns and rows are connected correctly. Unless there is chances for Blue smoke." %}

Here I chose a 2D array (Python List) to  save name of product, RFID unique number, and price of product respectively.
<pre class="line-numbers" data-start="14">
<code class="language-python">
{% raw %}
product = [
    ['Product1', 858624605840, 8000],
    ['Product2', 236811894460, 1000],
    ['Product3', 650705026722, 1200],
    ['Product4', 99658219146, 800],
    ['Product5', 30953647075, 900]
    ]
{% endraw %}
</code>
</pre>
{% include MyNote.html note_type="info" span_note="Info: " text="Here I added only 5 cards(white tags not blue keychain ring). You have to add details of RFID keychain rings . I written unique ID of each rings and cards on it using a cd marker. You have to just make a 2D array using that info as I written above." %}

As I mentioned in the class the GPIO pin connected to row of keypad  pulled-low to ground in order to avoid floating condition. GPIO pins connected to column is configured as output.

<pre class="line-numbers" data-start="21">
<code class="language-python">
{% raw %}
for row_pin in rows:
    GPIO.setup(row_pin, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)

for col_pin in cols:
    GPIO.setup(col_pin, GPIO.OUT)
{% endraw %}
</code>
</pre>
{% include MyNote.html note_type="info" span_note="Info: " text="Refer Raspberry Pi hardware reference <a href='https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2835/BCM2835-ARM-Peripherals.pdf'> documentation</a> in order to know which GPIO pins are pulled to LOW(Refer page number 102). If you have to change GPIO pins connected to keypad you have to keep this documentation in mind to find out GPIO pins pulled to ground."%}

The keypad is scanned by output a logical HIGH (1) through column pins one by one and scans for 0.3 seconds for key press in order to avoid noise as well as false key press. If there is a short circuit the raw GPIO pin gets a logical HIGH, so that we can detect key press. The following function helps to find out pressed key. Please go through 3 articles I given on [readme file](https://github.com/arunksoman/PCPL-electro/blob/master/README.md).

<pre class="line-numbers" data-start="26">
<code class="language-python">
{% raw %}
def get_key():
    key = 0
    for col_num, col_pin in enumerate(cols):
        GPIO.output(col_pin, 1)
        for row_num, row_pin in enumerate(rows):
            if GPIO.input(row_pin):
                key = keys[row_num][col_num]
        GPIO.output(col_pin, 0)
    return key
{% endraw %}
</code>
</pre>

`fixed` is a variable that stores maximum digit that can be used for transaction. Here I choose that as 5 so that we can do maximum transaction of ₹99999. `count` is the variable increments after each detection of each key press. `budget, count and fixed` are the variables used to convert each digits entered via key press into single number. `budget_enter` is a boolean variable used as an  _indicator_  to identify whether 5 times key press occurred or not.

<pre class="line-numbers" data-start="35">
<code class="language-python">
{% raw %}
count = 0
budget = 0
cart = 0
print("Enter your Budget as a 5 digit number on keypad:")
fixed = 5
Budget_enter = False
{% endraw %}
</code>
</pre>
{% include MyNote.html note_type="warning" span_note="Warning: " text="User have to enter budget as 5 digit number by prepending zeros to number. For example in order to set budget as 123, he should enter 00123"%}

Next line in the program just calls function within the Python package MFRC522. Please read MFRC522.py within directory MFRC522. This file is core of this project. I didn't discusses too much about this file due to lack of time. If you want know more [download](https://www.google.com/url?sa=t&source=web&rct=j&url=https://www.nxp.com/docs/en/data-sheet/MFRC522.pdf&ved=2ahUKEwjohfmAkNHhAhXG7nMBHeCKCL4QFjAAegQIAhAB&usg=AOvVaw1_QoS_CiF5TGZvT5_-zxo6){:target="_blank"} MFRC522 data sheet by NXP (New brand name of well known **Philips** in chip manufacturing industry). Go to page 36. There NXP discussed about more than 60 registers. Some of them are **reserved**. But most of them should be edited by using program. Description of these registers extended up to page 69 followed by commands to communicate with MFRC522 to RPi on page 70. This is too much longer so that I didn't discussed. But you don't want to bother about that, just scan data sheet provided by manufacture.
<pre class="line-numbers" data-start="41">
<code class="language-python">
{% raw %}
reader = SimpleMFRC522()
{% endraw %}
</code>
</pre>

First line calls function `get_key()`. If less than 6 key press occurred it converts key press one by one to 5 digit number.

<pre class="line-numbers" data-start="42">
<code class="language-python">
{% raw %}
while True:
    key = get_key()
    if count < 6:
        if key :
            print("count = ",count)
            count = count + 1
            temp = int(key) * (10 ** (fixed - count))
            budget = budget + temp
            print("budget = ", budget)
{% endraw %}
</code>
</pre>

If more than 5 key press occurred warn customer  and our indicator variable that indicates whether 5 digit entered or not(`Budget_enter`) made True.

<pre class="line-numbers" data-start="51">
<code class="language-python">
{% raw %}
             if count >= 5:
                Budget_enter = True
                print("You can't enter more than 5 digits")
{% endraw %}
</code>
</pre>

Only after getting 5 digits(or setting Budget) RFID cards are read.

<pre class="line-numbers" data-start="54">
<code class="language-python">
{% raw %}
    if Budget_enter:
        print("Hold a tag near the reader")
        id, text = reader.read() 
        print("ID: ", id)
        print(type(id))
{% endraw %}
</code>
</pre>
{% include MyNote.html note_type="warning" span_note="warning: " text="Since only after setting Budget RFID cards, the program reads RFID card, the customer those who are not interested to set budget should enter 5 zeros. Or you have to assign a key like A, B, C , D, *, or # to skip the keypad reading on that key press and directly enter in to loop given above. You can do something like this by adding stickers <b>Yes</b> and <b>No</b> on key A and key D respectively(The logic is given, but you have to enter this carefully on the program):<br />print('Are you interested to enter Budget?')<br/>if key == 'A':<br />&nbsp; &nbsp; &nbsp; &nbsp; ## Program to read Budget from keypad<br />If key == 'D': <br />&nbsp; &nbsp; &nbsp; &nbsp; count = 6 <br />The count > 6 automatically sets indicator variable <b>Budget_enter</b> to True"%}

This part of the program comparing RFID unique id stored on 2D list product. If find so, check the second index of 2D array to get prize of product. Add prize on the cart. If added up prize on the cart exceeds give a warning to customer. The program can terminate using keyboard interrupt `ctrl+c`. While terminating program the program automatically cleans up all last GPIO entries.

<pre class="line-numbers" data-start="59">
<code class="language-python">
{% raw %}
         for index in range(5):
            
            if id == product[index][1]:
                prize = product[index][2]
                print(index)
                print("prize = ", prize)
        cart = cart + prize
        print("cart = ", cart)
        if cart > budget:
            print("Warning: Your Budget Exceeded")
    time.sleep(0.3)
GPIO.cleanup()

{% endraw %}
</code>
</pre>

## How to remove a product from cart

In order to add and remove a product from cart one of you said an idea of adding a counter and on even RFID reading product should be added up with cart and on odd RFID reading product should be removed from cart. But in order to do so we want one counter for each card (total 10 counters) also for checking each reading we have to do a modulo division by 2 to find out number of reading is odd or even. My idea to avoid this counter by using a boolean value as an indicator (as we discussed on **Budget_enter** variable) to each product on the list as shown on example program below. Just copy and paste program given below and run the program on Thonny Python IDE. Choose id1, id2, id3 id4 or id5 on prompt. Observe what happened to list **product**  and **cart** . Enter same id again and observe the change in **product** and **cart** .

<pre class="line-numbers" data-start="1">
<code class="language-python">
{% raw %}
#Let a product worth 1300 on cart
cart = 1300
product = [
    ['product1', 'id1', 1200, False],
    ['product2', 'id2', 1300, False],
    ['product3', 'id3', 14000, False],
    ['product4', 'id4', 10000, False], 
    ['product5', 'id5', 1300, False]
    ]

id_select = input("Please Enter any id from id1 to id5: ")

for index in range(5):
	if id_select == product[index][1]:
		print("prize = ", product[index][2])
		if product[index][3] == False:
			product[index][3] = True
			## program to add product to cart
			cart = cart + product[index][2]
			print("Product added to cart =", cart)
print(product)
print("–---------------------------------------------------------------")
id_select = input("Enter same id again: ")
for index in range(5):
	if id_select == product[index][1]:
		print("prize = ", product[index][2])
		if product[index][3] == True:
			product[index][3] = False
			## program to remove product from cart
			cart = cart - product[index][2]
			print("Product removed from cart =", cart)

print(product)
{% endraw %}
</code>
</pre>
