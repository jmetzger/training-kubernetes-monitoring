# Service returns no data 

## Prepare 

  * Delete deployment

```
kubectl -n web-demo get pods -l app=nginx 
kubectl -n web-demo delete deploy nginx
 

```

## Setup Alert 

![image](https://github.com/user-attachments/assets/fcf54cce-0f2e-4e4a-8697-8f51de9388bb)

![image](https://github.com/user-attachments/assets/7c5b575d-0a80-4777-b3f3-53a1188f8720)

## Click on 

```
Preview and alert condition
```

## Safe and exit rules 

```
Safe rule and exit 
```

## Alert ausklappen und warten bis er feuert 

  1. Erst pending (dauert einen Moment)
  2. Dann firing und es kommt ein Benachrichtigung per Slack 

