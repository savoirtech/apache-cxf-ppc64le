# Apache CXF on PPC64LE

<figure>
<img src="./assets/images/raptor-computing-systems-blackbird-power.jpg"
alt="POWER" />
</figure>

The upcoming release of Apache CXF 4.1 will establish strong build and
runtime compatibility with JVMs running Linux on POWER.  

## So which JVMs has Apache CXF been curated to work well upon PPC64LE?

- Adoptium Eclipse Temurin

- IBM Semeru

- RedHat OpenJDK

These are the three distributions which provide PPC64LE support. 

Asoprium Eclipse Temurin and RedHat OpenJDK are both OpenJDK based, with
IBM Semuru using OpenJ9.

## What is special about running on PPC64LE?

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><figure>
<img src="./assets/images/x64.png" alt="x64" />
</figure></td>
<td style="text-align: left;"><figure>
<img src="./assets/images/arm.png" alt="arm" />
</figure></td>
<td style="text-align: left;"><figure>
<img src="./assets/images/power.png" alt="power" />
</figure></td>
</tr>
<tr>
<td style="text-align: left;"><p>x86-64 instruction set, commonly
supporting SMT, Efficient and Perforamnce cores available on some
implementations.</p></td>
<td style="text-align: left;"><p>ARM cores providing reduced instruction
sets. Efficient and Perforamnce cores available on some
implementations.</p></td>
<td style="text-align: left;"><p>POWER cores with optional use of SMT4
or SMT8 for greater thread count.</p></td>
</tr>
<tr>
<td style="text-align: left;"><p>Mobile, Desktop, and Server
oriented.</p></td>
<td style="text-align: left;"><p>Mobile oriented, with Server
implementations becoming more common.</p></td>
<td style="text-align: left;"><p>Workstation, Server oriented.</p></td>
</tr>
</tbody>
</table>

## Let’s take a look through the journey towards stable support.

| Jira Entry | Errata |
|----|----|
| [CorbaConduitTest no longer requires IBM JDK destination activation routine.](https://issues.apache.org/jira/browse/CXF-8994) | Older versions of IBM Java would require destinations to be activated ahead of test cases, that is no longer the case, as such we removed this behavoir. |
| [JAXRS Bean introspection utility Beanspector relies on Class.getMethods natural order.](https://issues.apache.org/jira/browse/CXF-8996) | We discovered that JAXRS Bean introspection utility Beanspector relies on Class.getMethods natural order. For most JVMs the Beanspector Tests will pass, however IBM Semeru does not return methods in the same ordering. Note: Class.getMethods does not provide a prescribed ordering of methods. We applied ordering to methods such that all JVMs tested would achieve the same results. |
| [AbstractSTSTokenTest and TransportBindingTests no longer need to set https protocol to TLSv1 on IBM Java.](https://issues.apache.org/jira/browse/CXF-8997) | IBM JDKs disable TLSv1 by default since around Java 8 (<https://community.ibm.com/community/user/wasdevops/blogs/hiroko-takamiya1/2021/06/19/ibm-java-80630-disables-tlsv1-tlsv11-by-default-ho>). Removing the test case IBM control flag allows the default TLS to pass the tests. |
| [KerberosTokenTest testKerberosViaCustomTokenAction should not run on IBM Java.](https://issues.apache.org/jira/browse/CXF-8999) | The test case fails on ClassNotFound com.ibm.security.jgss.InquireType - this is thrown due to wss4j-ws-security-common having a hard coded check for IBM Java to use the above mentioned class. A future improvement would be to update wss4j-ws-security-common to be IBM Semeru friendly, then update CXF accordingly. |
| [JAXRSMultithreadedClientTest test cases failing on IBM JDK.](https://issues.apache.org/jira/browse/CXF-9002) | This is was addressed via updates for Multi threading stability in other cards. |
| [TrustedAuthorityValidatorCRLTest#testIsCertChainValid fails when using Red Hat OpenJDK on PPC64LE.](https://issues.apache.org/jira/browse/CXF-9006) | This card required updating the certificates stored in the system tests folder. |
| [org.apache.cxf.systest.ws.action.SignatureWhitespaceTest test fail on RH OpenJDK.](https://issues.apache.org/jira/browse/CXF-9014) | Certificated used in the system test was updated from 1024-bit RSA key (weak) to RSA 2048/sha256. |

In our builds towards stable PPC64LE support, Apache CXF 4.1 will ship
with its internal performance script.  Using this script we’ve been able
to run JAX-RS, and JAX-WS workflows to help stress the JVM, and identify
runtime issues.
