Ở đây mình không nhắc đến Fail2band, vì nó là con dao 2 lưỡi(nếu tài nguyên của bạn không nhiều, Fail2band có thể làm chết hệ thống, trước khi hệ thống bị chết vì tấn công)
 
 Đổi port ssh
  - Trước khi đóng port 22 hãy mở thêm 1 port khác VD: 2244
    Sửa file /etc/ssh/sshd_config
    
    Port 2244
    
    Port 22
    
  - Cấu hình chặn ssh bằng password 
  
    **Trước khi thực hiện bước này hãy chắc chắn rằng các tài khoản đã được cấu hình ssh bằng private key.**
    
    sửa lại tham số sau trong /etc/ssh/sshd_config
    
        PasswordAuthentication no
        
note: Bgio sẽ có 2 port để ssh

cấu hình xong nhớ restart ssh

- Cấu hình iptable
    - Tránh tình trạng "tự sát" khi cấu hình iptable, cần allow ip whitelist
                
            -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
            -A INPUT -p icmp -j ACCEPT
            -A INPUT -i lo -j ACCEPT
            #allow ip whitelist
            -A INPUT -s x.x.x.a -j ACCEPT
            -A INPUT -s x.x.x.b -j ACCEPT
            -A INPUT -s x.x.x.c -j ACCEPT
            -A INPUT -s x.x.x.d -j ACCEPT
            -A INPUT -s x.x.x.e -j ACCEPT 
            #trong 5 phút chỉ cho phép 4 gói TCP/SYN từ 1 IP đến port 22 
            -A INPUT -p tcp -m tcp --dport 22 -m state --state NEW -m recent --set --name DEFAULT --rsource
            -A INPUT -p tcp -m tcp --dport 22 -m state --state NEW -m recent --update --seconds 300 --hitcount 4 --name DEFAULT --rsource -j DROP
            #Mở port ssh
            -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
            -A INPUT -p tcp -m state --state NEW -m tcp --dport 2244 -j ACCEPT
    
    
 Note: Việc cấu hình allow ip whitelist này sẽ tránh được tình trạng vì lí do nào đó cấu hình sai, iptable sẽ DROP tất các cả IP.  

Nhớ restart iptable

Bgio sẽ sử dụng 1 module của Linux PAM (Pluggable Authentication Modules) để band account ssh fail >3 lần.

mở file /etc/pam.d/system-auth

    auth        required      pam_faillock.so preauth silent audit deny=3 unlock_time=300
    
    auth        [default=die]  pam_faillock.so  authfail  audit  deny=3  unlock_time=300`
    account     required      pam_faillock.so

mở file /etc/pam.d/password-auth
    
    auth        required      pam_faillock.so preauth silent audit deny=3 unlock_time=300
    
    auth        [default=die]  pam_faillock.so  authfail  audit  deny=3  unlock_time=300
    account     required      pam_faillock.so
    
Các tham số   
 
   - audit – enables user auditing
    
   - deny – used to define the number of attempts
    
   - unlock_time – sets the time band account
    
    
**chú ý: thứ tự các dòng rất quan trọng. nếu cấu hình sai tất các user có thể bị lock.**

sau khi cấu hình xong 2 file sẽ giống thế này: 
![image](https://user-images.githubusercontent.com/19284401/130925653-c2fa58c9-efd8-44d4-8488-314b0d2e1ab8.png)

sau khi cấu hình xong restart lại sshd

check faillock
    
    faillock # danh sach tất các user đã bị log
    
    fail --reset #clear toàn bộ các tài khoản bị lock
    
    faillock --user abx --reset  clear 1 tài khoản
    
    
Lock password của users
        
        passwd -l root
        
        passwd -S root # kiểm tra lại tài khoản
        
![image](https://user-images.githubusercontent.com/19284401/130927930-a9bd918f-4d71-42f2-9972-01969970a162.png)

Cuối cùng hãy đóng port 22 trong /etc/ssh/sshd_config và iptable. Làm xong nhớ restart lại iptable và sshd
    
    
       




   
