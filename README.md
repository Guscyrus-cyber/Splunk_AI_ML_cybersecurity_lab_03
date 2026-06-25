AI/ML Cybersecurity Lab 03 – User and Entity Behavior Analytics (UEBA)\
\
Introduction
=======================================================================

In this lab, I used Splunk Enterprise to investigate user login behavior and identify potentially suspicious activity using User and Entity Behavior Analytics (UEBA) concepts. Unlike traditional security monitoring that focuses on network traffic or malware, UEBA focuses on understanding how users normally behave and detecting deviations from their established behavioral patterns.

The dataset contains user login activity, login times, geographic locations, device information, VPN usage, download activity, failed login attempts, and baseline behavioral information. By comparing current activity against a user's normal behavior, I can identify anomalies that may indicate account compromise, insider threats, unauthorized access, or other security incidents.

## **Objectives**

The objectives of this lab are to:

- Analyze user login behavior.

- Identify logins occurring at unusual times.

- Detect logins from new countries.

- Investigate the use of new devices.

- Examine VPN usage anomalies.

- Compare user activity against established baselines.

- Identify high-risk users through behavioral analysis.

## **AI/ML Concept**

This lab introduces User and Entity Behavior Analytics (UEBA), one of the most common applications of artificial intelligence and machine learning in modern Security Operations Centers (SOCs).

The primary AI/ML concepts used in this lab include:

- Behavioral Analytics

- User Profiling

- Anomaly Detection

- Risk Scoring

- Baseline Deviation Analysis

Rather than relying solely on signatures or known attack indicators, UEBA systems learn what normal user behavior looks like and identify activities that significantly deviate from those patterns.

## **Real-World Relevance**

UEBA ( User and Entity Behavior Analytics) capabilities are widely used in enterprise security platforms to detect account compromise, insider threats, credential abuse, and suspicious user activity. Similar techniques are used by modern security products such as Splunk User Behavior Analytics, Microsoft Sentinel UEBA, and Exabeam.

In this lab, I will use Splunk Enterprise to investigate user behavior and identify anomalous activities that would warrant further security investigation.

## **Dataset**

File:

ueba_dataset.csv

Fields:

user

login_hour

country

new_device

vpn

bytes_downloaded

failed_logins

baseline_country

baseline_login_hour

label

\
Uploading File & Verification\
\
\
\
\
I uploaded the ueba_dataset.csv dataset into Splunk Enterprise and verified that **50 events** were successfully indexed. For demonstration purposes, I only included a screenshot of the **first 5 events** displayed in Splunk. The full dataset contains 50 events and will be used throughout this lab to investigate user behavior patterns, identify anomalies, and perform User and Entity Behavior Analytics (UEBA) investigations.

**Which users logged in during unusual overnight hours?**

Using this query:\
\
index=auth sourcetype=ueba\
\| where login_hour \< 6\
\| table user login_hour country new_device vpn failed_logins label

\
\
\
\
I searched for login activity occurring during overnight hours between 1:00 AM and 5:00 AM. Organizations commonly monitor these hours because legitimate user activity is typically lower and attackers often operate during off-hours to avoid detection.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The users who logged in during unusual overnight hours were:

finance_admin\
backup_admin\
admin

All returned events were classified as Anomaly

I observed that all overnight login activity was associated with anomaly events. Additional suspicious indicators included:

Login times between 1 AM and 5 AM.\
New devices being used.\
VPN not being used.\
Elevated failed login counts.\
Logins originating from Russia, China, or Unknown locations.

These characteristics suggest behavior that deviates significantly from normal user activity and warrants further investigation.

**AI/ML Insight**

This is a classic UEBA detection. The model is not simply looking for:

login_hour \< 6

Instead it combines multiple behaviors:

| **Feature**   | **Observation**          |
|---------------|--------------------------|
| Login Hour    | 1–5 AM                   |
| Country       | Russia / China / Unknown |
| New Device    | Yes                      |
| VPN           | No                       |
| Failed Logins | High                     |
| Label         | Anomaly                  |

**. Which users logged in from new countries?**

Using this query:\
\
\
index=auth sourcetype=ueba\
\| where country != baseline_country\
\| table user country baseline_country login_hour new_device vpn label

\
\
I compared the user's current login country against their baseline country to identify logins originating from unexpected geographic locations. UEBA systems commonly use this technique to detect account compromise, credential theft, and suspicious remote access activity.
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The following users logged in from countries different from their normal baseline location:

| **User**      | **Country** | **Baseline Country** |
|---------------|-------------|----------------------|
| finance_admin | Russia      | USA                  |
| backup_admin  | China       | USA                  |
| admin         | Unknown     | USA                  |

All returned events were classified as: Anomaly

I identified three users whose login locations deviated from their established baseline country. Additional suspicious indicators included:

**.** Login activity during overnight hours.\
**.** New devices being used.\
**.** VPN disabled.\
**.** Elevated failed login counts.\
**.** Repeated logins from Russia, China, and Unknown locations.

The combination of geographic deviation and behavioral anomalies significantly increases the risk associated with these accounts.

**AI/ML Insight**

This is one of the most common UEBA detections in commercial products.

Normal behavior:

User → USA

Observed behavior:

finance_admin → Russia\
backup_admin → China\
admin → Unknown

A UEBA model learns:

Normal Country = USA

and then flags:

Country ≠ Baseline Country

as a behavioral deviation.

However, the location change alone is not enough. The model increases risk because the same users also exhibit:

. Unusual login hours\
. New devices\
. No VPN usage\
. High failed login counts

This correlation of multiple behavioral anomalies is what makes UEBA effective.

. **Which users used a new device?**

Using this query:\
\
index=auth sourcetype=ueba new_device="Yes"\
\| table user country login_hour vpn failed_logins label

\
\
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

I searched for users who authenticated from a new device. New device usage is an important UEBA indicator because attackers frequently access compromised accounts from systems that have never been observed for that user before.\
\
The users who used new devices were **finance_admin**, **backup_admin**, and **admin**. All associated events were classified as anomalies and were accompanied by other suspicious behaviors such as unusual login times, foreign or unknown locations, disabled VPN usage, and elevated failed login counts.

All returned events were classified as: Anomaly

I observed that every login from a new device shared several additional risk indicators:

| **Indicator** | **Observation**        |
|---------------|------------------------|
| New Device    | Yes                    |
| VPN           | No                     |
| Login Hour    | 1 AM – 5 AM            |
| Country       | Russia, China, Unknown |
| Failed Logins | 18–35                  |
| Label         | Anomaly                |

The repeated appearance of these indicators suggests that the new device activity is not isolated and should be treated as suspicious.

**AI/ML Insight**

A UEBA platform does not trigger an alert simply because:

new_device = Yes

Many legitimate users receive new laptops.

Instead, the risk increases when multiple behavioral indicators occur together:

New Device\
+\
Foreign Country\
+\
Overnight Login\
+\
No VPN\
+\
High Failed Logins

This correlation of behaviors is what makes UEBA one of the most powerful AI/ML technologies used in modern SOC environments.

**Which users deviated from their normal behavior?**

Using this query:\
\
index=auth sourcetype=ueba label=Anomaly\
\| table user login_hour baseline_login_hour country baseline_country new_device vpn failed_logins


\
\
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

I reviewed all events classified as anomalies and compared current user behavior against established baselines. The investigation focused on login times, geographic locations, device usage, VPN activity, and failed login attempts to identify users whose behavior significantly deviated from normal patterns.

The users who deviated from their normal behavior were:

**finance_admin**\
**backup_admin**\
**admin**

**Findings**

**finance_admin**

| **Behavior**  | **Baseline**  | **Observed** |
|---------------|---------------|--------------|
| Login Hour    | 09:00         | 01:00–05:00  |
| Country       | USA           | Russia       |
| Device        | Known Device  | New Device   |
| VPN           | Normally Used | Not Used     |
| Failed Logins | Low           | 25–35        |

**backup_admin**

| **Behavior**  | **Baseline**  | **Observed** |
|---------------|---------------|--------------|
| Login Hour    | 09:00         | 01:00–04:00  |
| Country       | USA           | China        |
| Device        | Known Device  | New Device   |
| VPN           | Normally Used | Not Used     |
| Failed Logins | Low           | 20–24        |

**admin**

| **Behavior**  | **Baseline**  | **Observed** |
|---------------|---------------|--------------|
| Login Hour    | 09:00         | 02:00–04:00  |
| Country       | USA           | Unknown      |
| Device        | Known Device  | New Device   |
| VPN           | Normally Used | Not Used     |
| Failed Logins | Low           | 18–21        |

**Conclusion**

I identified three users whose behavior deviated substantially from their established baseline patterns. The deviations included:

Overnight login activity\
New geographic locations\
New devices\
VPN avoidance\
Elevated failed login attempts

**AI/ML Insight**

This is exactly what UEBA platforms are designed to detect.

UEBA asks:

Is this user behaving differently than usual?

For example:

Normal:\
USA\
09:00\
Known Device\
VPN Enabled

versus:

Observed:\
Russia\
02:00\
New Device\
VPN Disabled\
30 Failed Logins

The more deviations observed, the higher the user's risk score becomes.

. **Which users present the highest risk?**

Using the query:**\
\**
index=auth sourcetype=ueba label=Anomaly\
\| stats count by user\
\| sort - count


\
\
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

I counted the number of anomaly events associated with each user and sorted the results in descending order. Users generating multiple anomaly events may represent a higher risk because suspicious behavior is recurring rather than isolated.

The highest-risk users identified in the dataset were:

**finance_admin** 4 anomaly events\
**admin** 3 anomaly events\
**backup_admin** 3 anomaly events

I observed that **finance_admin** generated the highest number of anomaly events in the dataset.

The account was associated with:

Login activity between 1 AM and 5 AM\
Logins originating from Russia\
New device usage\
VPN disabled\
High failed login counts\
Multiple deviations from baseline behavior

Similarly, **admin** and **backup_admin** exhibited repeated anomalous behavior and should also be prioritized for investigation.

In this lab, I used Splunk Enterprise to perform User and Entity Behavior Analytics (UEBA) on a dataset containing 50 user activity events. The objective was to identify behavioral anomalies by comparing current user activity against established baseline behavior. Through analysis of login times, geographic locations, device usage, VPN activity, and failed login attempts, I identified several users whose behavior significantly deviated from normal patterns.

**Key Findings**

- 50 total events analyzed

- 40 Normal events

- 10 Anomaly events

- Highest-risk user: **finance_admin**

- Additional high-risk users: **admin** and **backup_admin**

- Suspicious countries: **Russia**, **China**, and **Unknown**

- Suspicious login hours: **1 AM–5 AM**

- New device usage observed in all anomaly events

- VPN disabled in all anomaly events

- Elevated failed login counts associated with anomaly events

**AI/ML Concepts**

User and Entity Behavior Analytics (UEBA)\
Behavioral Analytics\
Anomaly Detection\
User Profiling\
Baseline Analysis\
Risk Scoring\
Behavior Deviation Detection
