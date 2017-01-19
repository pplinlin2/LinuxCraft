# Symmetric cryptography
## OpenSSL enc
使用DES3把test.txt加密成cipher.txt，再還原回plain.txt，
* -p: print out the key and IV used
```console
# echo "Hello world" > test.txt
# openssl enc -p -des3 -pass pass:123456 -in test.txt -out cipher.txt
salt=CDD8AEB4982C777D
key=301E28C706083DDE039DD9D7E988E055A531E4DB1575D325
iv =168537F8CB97F45D
# openssl enc -d -des3 -pass pass:123456 -in cipher.txt -out plain.txt
# hexdump -C cipher.txt
00000000  53 61 6c 74 65 64 5f 5f  cd d8 ae b4 98 2c 77 7d  |Salted__.....,w}|
00000010  13 fb 4f dd 19 fb f3 44  e3 16 86 28 08 09 52 d1  |..O....D...(..R.|
00000020
# hexdump -C plain.txt
00000000  48 65 6c 6c 6f 20 77 6f  72 6c 64 0a              |Hello world.|
0000000c
```
用DES的話可以看出來key的長度為DES3的三分之一
```console
# openssl enc -p -des -pass pass:123456 -in test.txt -out cipher.txt
salt=77A9B9FF0758DA06
key=A4A541ECD00A5AA3
iv =B2B2D44D82594DFA
```
