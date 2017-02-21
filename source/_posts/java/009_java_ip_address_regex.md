---
title: Java IP地址的正则表达式
date: 2017-02-21 15:14:41
tags:
categories: "Java学习笔记"
---

IP Address Regular Expression Pattern

```java
^([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\.([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\.
([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\.([01]?\\d\\d?|2[0-4]\\d|25[0-5])$
```

Description

```sh
^		#start of the line
 (		#  start of group #1
   [01]?\\d\\d? #    Can be one or two digits. If three digits appear, it must start either 0 or 1
		#    e.g ([0-9], [0-9][0-9],[0-1][0-9][0-9])
    |		#    ...or
   2[0-4]\\d	#    start with 2, follow by 0-4 and end with any digit (2[0-4][0-9])
    |           #    ...or
   25[0-5]      #    start with 2, follow by 5 and ends with 0-5 (25[0-5])
 )		#  end of group #2
  \.            #  follow by a dot "."
....            # repeat with 3 times (3x)
$		#end of the line
```

Whole combination means, digit from 0 to 255 and follow by a dot “.”, repeat 4 time and ending with no dot “.” Valid IP address format is “0-255.0-255.0-255.0-255”.

#### Java Regular Expression Example

```java

package com.viking.regex;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class IPAddressValidator{

    private Pattern pattern;
    private Matcher matcher;

    private static final String IPADDRESS_PATTERN =
                  		"^([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\." +
                  		"([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\." +
                  		"([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\." +
                  		"([01]?\\d\\d?|2[0-4]\\d|25[0-5])$";

    public IPAddressValidator(){
	     pattern = Pattern.compile(IPADDRESS_PATTERN);
    }

   /**
    * Validate ip address with regular expression
    * @param ip ip address for validation
    * @return true valid ip address, false invalid ip address
    */
    public boolean validate(final String ip){
  	  matcher = pattern.matcher(ip);
  	  return matcher.matches();
    }
}

```

<!--more-->

#### IP address that matches :

```sh
1. “1.1.1.1”, “255.255.255.255”,”192.168.1.1″ ,
2. “10.10.1.1”, “132.254.111.10”, “26.10.2.10”,
3. “127.0.0.1”
```

#### IP address that doesn’t match :

```sh
1. “10.10.10” – must have 4 “.”
2. “10.10” – must have 4 “.”
3. “10” – must have 4 “.”
4. “a.a.a.a” – only digit has allowed
5. “10.0.0.a” – only digit has allowed
6. “10.10.10.256” – digit must between [0-255]
7. “222.222.2.999” – digit must between [0-255]
8. “999.10.10.20” – digit must between [0-255]
9. “2222.22.22.22” – digit must between [0-255]
10. “22.2222.22.2” – digit must between [0-255]
```

#### Unit Test

```java
package com.viking.regex;

import org.testng.Assert;
import org.testng.annotations.*;

/**
 * IPAddress validator Testing
 * @author mkyong
 *
 */
public class IPAddressValidatorTest {

	private IPAddressValidator ipAddressValidator;

	@BeforeClass
        public void initData(){
		ipAddressValidator = new IPAddressValidator();
        }

	@DataProvider
	public Object[][] ValidIPAddressProvider() {
		return new Object[][]{
		   new Object[] {"1.1.1.1"},new Object[] {"255.255.255.255"},
                   new Object[] {"192.168.1.1"},new Object[] {"10.10.1.1"},
                   new Object[] {"132.254.111.10"},new Object[] {"26.10.2.10"},
		   new Object[] {"127.0.0.1"}
		};
	}

	@DataProvider
	public Object[][] InvalidIPAddressProvider() {
		return new Object[][]{
		   new Object[] {"10.10.10"},new Object[] {"10.10"},
                   new Object[] {"10"},new Object[] {"a.a.a.a"},
                   new Object[] {"10.0.0.a"},new Object[] {"10.10.10.256"},
		   new Object[] {"222.222.2.999"},new Object[] {"999.10.10.20"},
                   new Object[] {"2222.22.22.22"},new Object[] {"22.2222.22.2"},
                   new Object[] {"10.10.10"},new Object[] {"10.10.10"},
		};
	}

	@Test(dataProvider = "ValidIPAddressProvider")
	public void ValidIPAddressTest(String ip) {
		   boolean valid = ipAddressValidator.validate(ip);
		   System.out.println("IPAddress is valid : " + ip + " , " + valid);
		   Assert.assertEquals(true, valid);
	}

	@Test(dataProvider = "InvalidIPAddressProvider",
                 dependsOnMethods="ValidIPAddressTest")
	public void InValidIPAddressTest(String ip) {
		   boolean valid = ipAddressValidator.validate(ip);
		   System.out.println("IPAddress is valid : " + ip + " , " + valid);
		   Assert.assertEquals(false, valid);
	}
}

```

#### Unit Test – Result

```sh
IPAddress is valid : 1.1.1.1 , true
IPAddress is valid : 255.255.255.255 , true
IPAddress is valid : 192.168.1.1 , true
IPAddress is valid : 10.10.1.1 , true
IPAddress is valid : 132.254.111.10 , true
IPAddress is valid : 26.10.2.10 , true
IPAddress is valid : 127.0.0.1 , true
IPAddress is valid : 10.10.10 , false
IPAddress is valid : 10.10 , false
IPAddress is valid : 10 , false
IPAddress is valid : a.a.a.a , false
IPAddress is valid : 10.0.0.a , false
IPAddress is valid : 10.10.10.256 , false
IPAddress is valid : 222.222.2.999 , false
IPAddress is valid : 999.10.10.20 , false
IPAddress is valid : 2222.22.22.22 , false
IPAddress is valid : 22.2222.22.2 , false

===============================================
    com.viking.regex.IPAddressValidatorTest
    Tests run: 2, Failures: 0, Skips: 0
===============================================

===============================================

Total tests run: 2, Failures: 0, Skips: 0
===============================================

```

#### References

[How to validate IP address with regular expression](https://www.mkyong.com/regular-expressions/how-to-validate-ip-address-with-regular-expression/)

[IP address](https://en.wikipedia.org/wiki/IP_address)
