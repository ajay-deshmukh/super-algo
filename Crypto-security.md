## Cryptograpy and Network security algorithm(s)

### Ceasor Cipher
```python
def ecrypt_func(txt, s):
	result = ""
	for i in range(len(txt)):
		char = txt[i]
		if (char.isupper()):
			result += chr((ord(char) + s - 64) % 26 + 65)
		else:
			result += chr((ord(char) + s - 96) % 26 + 97)
	return result

txt = "Ajay"
s = 5

print("Plain  text: ", txt)
print("Shifting pattern: ", str(s))
print("Cipher text: ", ecrypt_func(txt, s))
```

### Deffie Hellman

```python
from random import randint

if __name__ == '__main__':

    P = 34
    
    G = 12
    
    print('The Value of P is :%d'%(P))
    print('The Value of G is :%d'%(G))
    
    a = 4
    print('The Private Key a for Alice is :%d'%(a))
    
    # gets the generated key
    x = int(pow(G,a,P))
    
    b = 3
    print('The Private Key b for Bob is :%d'%(b))
    
    # gets the generated key
    y = int(pow(G,b,P))
    
    
    # Secret key for Alice
    ka = int(pow(y,a,P))
    
    # Secret key for Bob
    kb = int(pow(x,b,P))
    
    print('Secret key for the Alice is : %d'%(ka))
    print('Secret Key for the Bob is : %d'%(kb))
```

### CElgamal algorithm

```python 
import random
from math import pow
a=random.randint(2,10)
#To fing gcd of two numbers
def gcd(a,b):
    if a<b:
        return gcd(b,a)
    elif a%b==0:
        return b
    else:
        return gcd(b,a%b)
#For key generation i.e. large random number
def gen_key(q):
    key= random.randint(pow(10,20),q)
    while gcd(q,key)!=1:
        key=random.randint(pow(10,20),q)
    return key
def power(a,b,c):
    x=1
    y=a
    while b>0:
        if b%2==0:
            x=(x*y)%c;
        y=(y*y)%c
        b=int(b/2)
    return x%c
#For asymetric encryption
def encryption(msg,q,h,g):
    ct=[]
    k=gen_key(q)
    s=power(h,k,q)
    p=power(g,k,q)
    for i in range(0,len(msg)):
        ct.append(msg[i])
    print("g^k used= ",p)
    print("g^ak used= ",s)
    for i in range(0,len(ct)):
        ct[i]=s*ord(ct[i])
    return ct,p
#For decryption
def decryption(ct,p,key,q):
    pt=[]
    h=power(p,key,q)
    for i in range(0,len(ct)):
        pt.append(chr(int(ct[i]/h)))
    return pt
msg=input("Enter message: ")
q=random.randint(pow(10,20),pow(10,50))
g=random.randint(2,q)
key=gen_key(q)
h=power(g,key,q)
print("g used=",g)
print("g^a used=",h)
ct,p=encryption(msg,q,h,g)
print("Original Message=",msg)
print("Encrypted Maessage=",ct)
pt=decryption(ct,p,key,q)
d_msg=''.join(pt)
print("Decryted Message=",d_msg)
```


### Hill cipher

```python 
keyMatrix = [[0] * 3 for i in range(3)]

messageVector = [[0] for i in range(3)]

cipherMatrix = [[0] for i in range(3)]

def getKeyMatrix(key):
	k = 0
	for i in range(3):
		for j in range(3):
			keyMatrix[i][j] = ord(key[k]) % 65
			k += 1

def encrypt(messageVector):
	for i in range(3):
		for j in range(1):
			cipherMatrix[i][j] = 0
			for x in range(3):
				cipherMatrix[i][j] += (keyMatrix[i][x] *
									messageVector[x][j])
			cipherMatrix[i][j] = cipherMatrix[i][j] % 26

def HillCipher(message, key):

	getKeyMatrix(key)
	for i in range(3):
		messageVector[i][0] = ord(message[i]) % 65
	encrypt(messageVector)
	CipherText = []
	for i in range(3):
		CipherText.append(chr(cipherMatrix[i][0] + 65))
	print("Ciphertext: ", "".join(CipherText))


def main():
	message = "INK"
	print(message," is a message")

	key = "KDFSKFBER"
	print(key, " is a key")

	HillCipher(message, key)

if __name__ == "__main__":
	main()
```


### Play flair cipher

```python 
key=input("Enter key")
key=key.replace(" ", "")
key=key.upper()
def matrix(x,y,initial):
    return [[initial for i in range(x)] for j in range(y)]
    
result=list()
for c in key: #storing key
    if c not in result:
        if c=='J':
            result.append('I')
        else:
            result.append(c)
flag=0
for i in range(65,91): #storing other character
    if chr(i) not in result:
        if i==73 and chr(74) not in result:
            result.append("I")
            flag=1
        elif flag==0 and i==73 or i==74:
            pass    
        else:
            result.append(chr(i))
k=0
my_matrix=matrix(5,5,0) #initialize matrix
for i in range(0,5): #making matrix
    for j in range(0,5):
        my_matrix[i][j]=result[k]
        k+=1

def locindex(c): #get location of each character
    loc=list()
    if c=='J':
        c='I'
    for i ,j in enumerate(my_matrix):
        for k,l in enumerate(j):
            if c==l:
                loc.append(i)
                loc.append(k)
                return loc
            
def encrypt():  #Encryption
    msg=str(input("ENTER MSG:"))
    msg=msg.upper()
    msg=msg.replace(" ", "")             
    i=0
    for s in range(0,len(msg)+1,2):
        if s<len(msg)-1:
            if msg[s]==msg[s+1]:
                msg=msg[:s+1]+'X'+msg[s+1:]
    if len(msg)%2!=0:
        msg=msg[:]+'X'
    print("CIPHER TEXT:",end=' ')
    while i<len(msg):
        loc=list()
        loc=locindex(msg[i])
        loc1=list()
        loc1=locindex(msg[i+1])
        if loc[1]==loc1[1]:
            print("{}{}".format(my_matrix[(loc[0]+1)%5][loc[1]],my_matrix[(loc1[0]+1)%5][loc1[1]]),end=' ')
        elif loc[0]==loc1[0]:
            print("{}{}".format(my_matrix[loc[0]][(loc[1]+1)%5],my_matrix[loc1[0]][(loc1[1]+1)%5]),end=' ')  
        else:
            print("{}{}".format(my_matrix[loc[0]][loc1[1]],my_matrix[loc1[0]][loc[1]]),end=' ')    
        i=i+2        
                 
def decrypt():  #decryption
    msg=str(input("ENTER CIPHER TEXT:"))
    msg=msg.upper()
    msg=msg.replace(" ", "")
    print("PLAIN TEXT:",end=' ')
    i=0
    while i<len(msg):
        loc=list()
        loc=locindex(msg[i])
        loc1=list()
        loc1=locindex(msg[i+1])
        if loc[1]==loc1[1]:
            print("{}{}".format(my_matrix[(loc[0]-1)%5][loc[1]],my_matrix[(loc1[0]-1)%5][loc1[1]]),end=' ')
        elif loc[0]==loc1[0]:
            print("{}{}".format(my_matrix[loc[0]][(loc[1]-1)%5],my_matrix[loc1[0]][(loc1[1]-1)%5]),end=' ')  
        else:
            print("{}{}".format(my_matrix[loc[0]][loc1[1]],my_matrix[loc1[0]][loc[1]]),end=' ')    
        i=i+2        

while(1):
    choice=int(input("\n 1.Encryption \n 2.Decryption: \n 3.EXIT"))
    if choice==1:
        encrypt()
    elif choice==2:
        decrypt()
    elif choice==3:
        exit()
    else:
        print("Choose correct choice")
```


### Rc4 algorothm

```python
MOD = 256


def KSA(key):
    
    key_length = len(key)
    # create the array "S"
    S = list(range(MOD))  # [0,1,2, ... , 255]
    j = 0
    for i in range(MOD):
        j = (j + S[i] + key[i % key_length]) % MOD
        S[i], S[j] = S[j], S[i]  # swap values

    return S


def PRGA(S):
    
    i = 0
    j = 0
    while True:
        i = (i + 1) % MOD
        j = (j + S[i]) % MOD

        S[i], S[j] = S[j], S[i]  # swap values
        K = S[(S[i] + S[j]) % MOD]
        yield K


def get_keystream(key):
    
    S = KSA(key)
    return PRGA(S)


def encrypt_logic(key, text):
    
    # For plaintext key, use this
    key = [ord(c) for c in key]
    # If key is in hex:
    # key = codecs.decode(key, 'hex_codec')
    # key = [c for c in key]
    keystream = get_keystream(key)

    res = []
    for c in text:
        val = ("%02X" % (c ^ next(keystream)))  # XOR and taking hex
        res.append(val)
    return ''.join(res)


def encrypt(key, plaintext):
    
    plaintext = [ord(c) for c in plaintext]
    return encrypt_logic(key, plaintext)


def decrypt(key, ciphertext):
    
    ciphertext = codecs.decode(ciphertext, 'hex_codec')
    res = encrypt_logic(key, ciphertext)
    return codecs.decode(res, 'hex_codec').decode('utf-8')


def main():

    key = 'not-so-random-key'  # plaintext
    plaintext = 'implementation is correct'  # plaintext
    # encrypt the plaintext, using key and RC4 algorithm
    ciphertext = encrypt(key, plaintext)
    print('plaintext:', plaintext)
    print('ciphertext:', ciphertext)
    # ..
    # Let's check the implementation
    # ..
    ciphertext = '2D7FEE79FFCE80B7DDB7BDA5A7F878CE298615'\
        '476F86F3B890FD4746BE2D8F741395F884B4A35CE979'
    # change ciphertext to string again
    decrypted = decrypt(key, ciphertext)
    print('decrypted:', decrypted)

    if plaintext == decrypted:
        print('\nfind it, correct.')
    else:
        print('done ! ')

    # until next time folks !


def test():

    assert(encrypt('Key', 'Plaintext')) == 'BBF316E8D940AF0AD3'
    assert(decrypt('Key', 'BBF316E8D940AF0AD3')) == 'Plaintext'
    assert(encrypt('Wiki', 'pedia')) == '1021BF0420'
    assert(decrypt('Wiki', '1021BF0420')) == 'pedia'
    assert(encrypt('Secret',
                   'Attack at dawn')) == '45A01F645FC35B383552544B9BF5'
    assert(decrypt('Secret',
                   '45A01F645FC35B383552544B9BF5')) == 'Attack at dawn'

if __name__ == '__main__':
    main()
```

### rsa algorithm

```python
from decimal import Decimal
def gcd(k,totient):
	while totient!= 0:
		c = k % totient
		k = totient
		totient= c
		return k
#input variables
d=0
p = int(input("prime no. p:"))
q = int(input("prime no. q: "))
message = int(input("numric message: "))
#calculate n
n = p*q
#calculate totient
totient = (p-1)*(q-1)
#calculate K
for k in range(2,totient):
	if gcd(k,totient)== 1:
		break

for i in range(1,10):
	x = 1 + i*totient
	if x % k == 0:
		d = int(x/k)
		break
local_cipher = Decimal(0)
local_cipher =pow(message,k)
cipher_text = local_cipher % n

decrypt_t = Decimal(0)
decrypt_t= pow(cipher_text,d)
decrpyted_text = decrypt_t % n

print('modulus of n = '+str(n))
print('co-prime k = '+str(k))
print('totient function = '+str(totient))
print('modulus of decrypt fn d = '+str(d))
print('cipher text = '+str(cipher_text))
print('decrypted text = '+str(decrpyted_text))
```
