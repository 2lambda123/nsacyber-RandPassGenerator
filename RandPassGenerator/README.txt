RandPassGenerator 1.3.2

The RandPassGenerator Java application is a simple command-line utility for generating random passwords, passphrases, and raw hex keys.  It is designed very conservatively to ensure that the random values it provides offer full cryptographic strength requested by the user.

USAGE INFORMATION

To use RandPassGenerator, you'll need the Oracle Java Runtime Environment; any recent version should be sufficient, but at a minimum version 9 is recommended.

The RandPassGenerator can also run from a terminal or console.
The command-line syntax is simple:

	java -jar RandPassGenerator.jar  [options]

The following options are accepted:
  
Common options:
  
-v	   Print verbose messages during operation, in addition to logging
  
-str S     Use generation strength of S bits (default: 160)
  
-pw N	   Generate N random password of the specified strength
 
-pp N	   Generate N random passphrases of the specified strength
  
-k N	   Generate N random keys of the specified strength
 
-enc	   Encrypt generated random key using a random password that is at least a 16 characters (256-bit AES) and write to file named the Key ID (KEY_ID.enc). A prompt for a random password to us will appear. Users should generate a random password to use for encryption prior to generating keys. ("java -jar RandPassGenerator.jar -pw 1 -str 96" will generate a 16 character password).

-decrypt   Decrypt encrypted key file using a random password that is at least a 16 characters and save as text file (KEY_ID_decrypted.txt). A prompt for the name of the encrypted file to decrypt will appear, then a prompt for the random password to use will appear. 

Unusual options:
  
-pplen M  When generating passphrases, longest word should be M letters long
 (minimum value of M is 3) 
 
-ppurl U  Use the URL U to load words for passphrase (default: use internal list)	
  
-pwcs P   Use character pattern P for characters to use in passwords
 (lowercase letter, uppercase letter, number, special character, or combination) 
-log F    Log all operations to the log file F (default: ./randpass.log)
   
-out F    Write output to file F (default: writes to stdout)
  
-c N 	  Format output passwords and keys in chunks of N characters
  
-sep S    For chunk formatting, use S as the separator (default: -)

-rcc N    For passphrases, randomly upcase first N letters of each word (default: 0)
  
At least one of the options -pw, -pp, or -k must be supplied.  The keys, passwords, or passphrases produced by RandPassGenerator will be written to the standard output (stdout), so they can easily be redirected to a file. The -out option can also be used to write the output to a file.  All messages are written to the standard error (stderr).  

Detailed log messages are appended to the specified log file - if the log file cannot be opened, then the tool will not run. 

Note that the -pwcs option is a little strange.  Each character in the value represents a full set of characters.  Any lowercase letter
means "add a character set of all lowercase letters", any uppercase letter means "add a set of all uppercase letter", any digit means
"add a set of all digits", and any thing else means "add a set of all punctuation marks".  There is no way to supply fully custom character set.  Normally, you should not use the -pwcs option, you should let RandPassGenerator use its default character set.


Below are some examples of running RandPassGenerator in normal ways:

Example 1: generate 5 random passwords using the default mixed character set, at default strength of 160, saved into file GoodPasswords.dat

    java -jar RandPassGenerator.jar -pw 5  >GoodPasswords.dat

Example 2: generate 20 random passphrases using the default dictionary, at strength of 256, with verbose messages, using words up to 9 letters long, and output saved into the file passphrases.txt

    java -jar RandPassGenerator.jar -v -pp 20 -str 256 -pplen 9 >passphrases.txt

Example 3: generate 200 random keys at strength of 192, with logging to keygen.log, and output to mykeys.out.

    java -jar RandPassGenerator.jar -k 200 -str 192 -log keygen.log -out mykeys.out

Example 4: generate 100 passwords at strength 160, using a character set of lowercase letters and digits, with output redirected to hi-quality-stuff.txt

    java -jar RandPassGenerator.jar -pw 100 -pwcs "a0"  >hi-quality-stuff.txt

Example 5: generate 10 passwords at strength 128, formatted into chunks of five characters each, separated by /.

    java -jar RandPassGenerator.jar -pw 10 -str 128 -c 5 -sep /
    
Example 6: generate 1 random key at strength 256, and encrypt to file using random password.

    java -jar RandPassGenerator.jar -k 1 -str 256 -enc

Example 7: Decrypt encrypted key file.  java -jar RandPassGenerator.jar -decrypt

DESIGN INFORMATION

The foundation of RandPassGenerator is an implementation of the NIST SP800-90 HashDRBG.  It uses entropy, carefully gathered from system sources, to generate quality random output.  The internal strength of the DRBG is 192 bits, according to NIST SP800-57, using the SHA-384 algorithm. In accordance with SP800-90, the DRBG is seeded with at least 888 bits of high quality entropy from entropy sources prior to any operation.
 
This implementation uses the seed mechanism of the Java SecureRandom class for gathering entropy. This implementation performs self-tests at every execution, so that users can be confident that no library problems have affected operation. Two kinds of self-tests are performed:
    
1. Known-answer tests from the NIST Hash_DRBG verification suite test file.
    
2. Simple statistical tests on DRBG output.

If the tests don't pass, the tool reports failure and refuses to run.

The strength mechanism implemented here is quite simple.  For passwords, the size of the character set used defines the bits-per-character, and password length is then computed to meet or exceed the requested strength (typically, this is somewhere around 5-6 bits per character).  Similarly, for passphrases the size of the usable dictionary defines the bits-per-word, and passphrase length is then computed to meet or exceed the requested strength (for the default dictionary and settings, roughly 16 bits per word).Duplicates are eliminated and the entropy is computed based on the number of unique characters or words.

The RandPassGenerator tool performs extensive logging.  By default, log entries are appended to the local file "randpass.log".  No actual key data, random data, or seed data is written to the log file.


