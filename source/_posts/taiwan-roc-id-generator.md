---
title: 台湾身份证字号产生器
date: 2019-04-26 15:33:38
tags: [php]
categories: 后端
---

参考 [身份證字號產生器](https://people.debian.org/~paulliu/ROCid.html) 写了一份PHP版本
<!--more-->
## 源码

```php
/**
 * 台灣身份證字號生成 & 驗證
 * @example $id = (new TaiwanRocId)->generate();
 * @example $ok = (new TaiwanRocId)->validate($id);
 */
class TaiwanRocId
{
    const MALE   = 1; // 男
    const FEMALE = 2; // 女

    private $cityMaps = [
        'A' => 10, // 台北市
        'B' => 11, // 台中市
        'C' => 12, // 基隆市
        'D' => 13, // 台南市
        'E' => 14, // 高雄市
        'F' => 15, // 台北縣
        'G' => 16, // 宜蘭縣
        'H' => 17, // 桃園縣
        'J' => 18, // 新竹縣
        'K' => 19, // 苗栗縣
        'L' => 20, // 台中縣
        'M' => 21, // 南投縣
        'N' => 22, // 彰化縣
        'P' => 23, // 雲林縣
        'Q' => 24, // 嘉義縣
        'R' => 25, // 台南縣
        'S' => 26, // 高雄縣
        'T' => 27, // 屏東縣
        'U' => 28, // 花蓮縣
        'V' => 29, // 台東縣
        'X' => 30, // 澎湖縣
        'Y' => 31, // 陽明山
        'W' => 32, // 金門縣
        'Z' => 33, // 連江縣
        'O' => 35, // 新竹市
        'I' => 34, // 嘉義市
    ];

    /**
     * 生成身份證字號
     *
     * @param string $city 城市
     * @param integer $sex 性別
     * @param string $mid 流水號
     * @return void
     */
    public function generate(string $city = null, int $sex = null, string $mid = null)
    {
        $cities = array_keys($this->cityMaps);

        if (is_null($city)) {
            $index    = array_rand($cities);
            $city     = $cities[$index];
            $cityCode = $this->cityMaps[$city];
        } else {
            $city     = strtoupper($city);
            $cityCode = $this->cityMaps[$city] ?? '';
        }

        if (is_null($sex)) {
            $sex = rand(0, 1);
        }

        if (is_null($mid)) {
            $mid = $this->midRand();
        }

        // 驗證城市
        if (!in_array($city, $cities)) {
            throw new \Exception("Invalid city", 1);
        }

        // 驗證性別
        if (!in_array($sex, [self::MALE, self::FEMALE])) {
            throw new \Exception("Invalid sex", 1);
        }

        $verifyCode = $this->calAll($cityCode, $sex, $mid);

        return sprintf('%s%s%s%s', $city, $sex, $mid, $verifyCode);
    }

    /**
     * 驗證身份證字號
     *
     * @param string $id 身份證字號
     * @return bool
     */
    public function validate(string $id = '')
    {
        [$city, $sex, $mid, $verifyCode] = [
            substr($id, 0, 1),
            substr($id, 1, 1),
            substr($id, 2, -1),
            substr($id, -1, 1),
        ];

        return $verifyCode == $this->calAll($this->cityMaps[$city], $sex, $mid);
    }

    /**
     * 隨機流水號
     *
     * @return string
     */
    private function midRand()
    {
        return str_pad(rand(0, 9999999), 7, '0', STR_PAD_LEFT);
    }

    /**
     * 計算驗證碼
     *
     * @param int $city
     * @param int $sex
     * @param string|int $mid
     * @return int
     */
    private function calAll($city, $sex, $mid)
    {
        $ret = 0;
        $ret = $this->calCity($city) + $this->calSex($sex) + $this->calMid($mid);
        $ret = $ret % 10;
        $ret = 10 - $ret;
        $ret = $ret % 10;

        return $ret;
    }

    /**
     * 計算城市驗證碼
     *
     * @param int $city
     * @return int
     */
    private function calCity(int $city)
    {
        return substr($city, 0, 1) + substr($city, 1, 1) * 9;
    }

    /**
     * 計算性別驗證碼
     *
     * @param integer $sex
     * @return int
     */
    private function calSex(int $sex)
    {
        return $sex * 8;
    }

    /**
     * 計算流水驗證碼
     *
     * @param string|int $mid
     * @return int
     */
    private function calMid($mid)
    {
        $ret = 0;
        $len = strlen($mid);

        for ($i = 0; $i < $len; $i++) {
            $ret = $ret + (7 - $i) * substr($mid, $i, 1);
        }

        return $ret;
    }
}
```

## 例子

### 生成身份证字号

```php
$Roc = new TaiwanRocId;

// 随机生成
$id = $Roc->generate();

// 指定城市、性别
$id = $Roc->generate('A', 1);
```

### 验证合法性

```php
$ok = (new TaiwanRocId)->validate("A197651254"); // true
```