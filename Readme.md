#RaspberryPI 4B CentOS Setting with SSD
## CentOS7

급한 분들은 ★을 찾아서 보시면 됩니다.   
★순서 5 -> 1 -> 7   
핵심은 라즈비안으로 rpi-eeprom 등을 실행하고, elf파일과 dat파일만 CentOS로 복사하는 겁니다.   
따라서 SD는 라즈비안, SSD는 CentOS로 하시면 간단!

> 현 시점에서 CentOS 중 RaspberryPI를 지원하는 버전은 7버전이 유일하다.   
> 2020-01-11을 기준으로 CentOS7 1810 버전만 작동이 되었으나   
> 2020-08-12 현 시점을 기준으로 CentOS7 2003 버전도 적용이 가능하다   

> CentOS7.8.2003 버전(CentOS 7.8.2003 Version)   
[CentOS-Userland-7-armv7hl-RaspberryPI-Minimal-4-2003-sda.raw.xz](http://mirror.freethought-internet.co.uk/centos-altarch/7.8.2003/isos/armhfp/CentOS-Userland-7-armv7hl-RaspberryPI-Minimal-4-2003-sda.raw.xz)

Raspberry4의 경우엔 아쉽게도 CentOS GNOME 혹은 KDE버전은 사용할 수 없다.   
Minimal뒤에 4가 붙은 버전만 Raspberry4에서 사용이 가능하고 그 외에는 Raspberry3까지만 지원한다.   

* * *
## CentOS7 설치

필자의 경우엔 2가지 방법을 사용해서 CentOS를 설치하였는데, Rufus와 Raspberry PI Imager를 이용해서 시도해보았다.   

> Link for Rufus   
[Rufus](https://rufus.ie/)

> Link for Raspberry PI Imager   
[Raspberry PI Imager](https://www.raspberrypi.org/downloads/)

* * *
## SSD로 부팅하기 위한 시도 과정
### 1. 구글 참고 ★

사실상 라즈베리 파이를 처음 사용해보기 때문에 SD카드로 부팅하는 방법은 알았어도 SSD로 부팅하는 방법은 몰랐다.   
더하여 최신버전 라즈베리를 구매해버리는 바람에 소프트웨어도 불안정한 상태라는 이야기를 들어 더욱 방황했다.

그렇게 방황하던 중 찾은 자료가 이것이다.   
[Official Raspberry Pi 4 USB Boot from SSD (beta)](https://peyanski.com/official-raspberry-pi-4-usb-boot/)   

해당 사이트를 방문하면 굉장히 상세하게 SSD를 통한 부팅 방법을 알려주는데, 아쉽게도 라즈비안에 한정되어 있었다.

명령어 정리

```linux
sudo apt-get update
sudo apt-get upgrade -y
sudo rpi-update
sudo reboot
```

최우선 적으로 모든 패키지를 업데이트 해준 후 리부팅 해준다.   
재부팅 후 ssh에 다시 로그인하여 다음 커맨드를 실행한다.

> 설치되어 있는 경우도 있는데, 이럴 경우엔 바로 다음단계로 직진
```linux
sudo apt install rpi-eeprom -y
```

설치가 끝난 후 해당 커맨드를 입력해준다.
```linux
sudo sed -i 's/critical/beta/g' /etc/default/rpi-eeprom-update
```

RaspberryPI4의 경우 기존 2GB/4GB 모델은 초기 오류가 있었다고 한여 USB부팅이 제대로 구현되지 않은 상태였다고 한다.   
그렇기 때문에 아래의 명령어를 통해 부트로더를 업데이트 해주어야 한다. (8GB 모델의 경우엔 적용이 되어있는것으로 판단되나 확실치 않다.)   

> sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/beta/pieeprom-yyyy-MM-dd.bin

yyyy-MM-dd는 실제 펌웨어 출시 일자를 기준으로 하면 되는데, 현 시점을 기준으로 최선 펌웨어는 2020-07-31 버전으로 확인되었다.   
따라서 명령어는 다음과 같다.

```linux
sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/beta/pieeprom-2020-07-31.bin
```

해당 명령을 입력하면

> .bin   
> BCM2711 detected    
> Dedicated VL805 EEPROM detected   
> BOOTFS /boot   
> *** INSTALLING /lib/firmware/raspberrypi/bootloader/beta/pieeprom-2020-07-31.bin ***   
> BOOTFS /boot   
> EEPROM update pending. Please reboot to apply the update.   

다음과 같은 결과가 나올 것이다.

이렇게 표시된다면 바로 리붓 (sudo reboot)

재부팅 후 부트로더를 확인해야 하므로 다음 명령어를 실행한다.

```linux
vcgencmd bootloader_version
vcgencmd bootloader_config
```

해당 명령을 실행했을 때 확인해야 할 부분은 크게 두 가지이다.

첫번째는 bootloader_version을 실행했을 때   
> July 31 2020 --:--:--   
다음과 같은 결과를 나타내는지

두번째는 bootloader_config를 실행했을 때
> BOOT_ORDER=0xf41   
마지막 줄이 해당 결과와 같은지 확인하면 된다.

__0xf41은 4 : USB부팅 1 : SD카드 부팅 순서이기 때문에 USB->SD카드 순으로 부팅한다__

이후엔 라즈비안을 SSD에 올려 연결한 후 다음 명령어를 실행하면 SSD로 부팅할 수 있다고 했다.

```linux
sudo mkdir /mnt/mydisk
sudo mount /dev/sda1 /mnt/mydisk
sudo cp /boot/*.elf /mnt/mydisk
sudo cp /boot/*.dat /mnt/mydisk
```

mydisk디렉터리를 생성한 후 해당 디렉터리에 sda1을 마운트 한다   
sda1 = SSD의 첫번째 파티션   
이후 boot폴더에 .elf파일과 .dat파일을 모두 mydisk폴더에 복사 붙여넣기를 한다.

해당 과정이 끝난 후 SD카드를 제거하고 SSD로만 부팅을 해도 부팅이 된다고 한다.

> 실제로 시도해봤을 때 라즈비안은 잘 구동되었다!

* * *
## 하지만 우리의 목표는...

맞다 우리의 목표는 CentOS이다.   
사실 위의 과정을 설명한 이유는 해당 과정을 응용해서 CentOS를 구동했기 때문이다.

* * *
### 2. CentOS로 1과정 따라하기

> 결과는 뻘짓

당연히 라즈비안을 기반으로 했을 때 잘 되었기 때문에 CentOS로 해당과정을 따라하면 순탄할 것이라고 생각했다.   
~~어림도 없지~~

CentOS엔 rpi-eeprom패키지가 존재하지 않았고, 리눅스도 그렇게까지 많이 다루진 않았기에 상대할 방법이 없었다.   
기가 막히게도 bootloader는 최신버전으로 적용이 되어있었고, BOOT_ORDER도 0xf41로 맞춰져 있었기에   
dat와 elf만 복사해서 붙여넣고 구동을 했더니!

~~응 아니야 다시해~~

> 섣불리 따라하지 맙시다

* * *
### 3. 와! 1810 지원종료!

> 마찬가지로 2번째 뻘짓

처음엔 Minial-4가 뭔지도 모르고 현존하는 글들이 전부 1810버전 Minimal4를 사용하고 있었기 때문에   
당연히 필자도 2003버전이라서 안되는구나~ 하면서 1810을 찾아나섰는데... 기가 막히게 CentOS 지원종료 아카이브에 들어가계셨다.

어찌저찌 찾아내서 설치한 후 2번 과정을 반복해 보았는데...

~~응 다시해 돌아가~~

> 2번째 퇴짜 엔딩

* * *
### 4. 드디어 이해한 Minimal-4의 뜻

유심히 패키지 명을 관찰하고 있었는데, Minimal 버전과 Minimal-4버전은 RaspberryPI CentOS에만 존재했다.   
이에 아 Userland-7-raspberrypi-Minimal-4가 라즈베리4전용 CentOS구나를 드디어 깨달았다.   
~~어쩌면 필자가 멍청이일지도...~~

~~물론 이해하기 전까지 SD카드로 어떻게 부팅시킨건지는 필자도 아직도 미스테리~~

어찌되었든 이제 돌파구가 보여 SSD로 시도해봤다!

~~다시 돌아가~~

> 라즈베리한테 3번째 퇴짜맞은 썰 푼다

* * *
### 5. 꼼수 ★

라즈베리에 꼬박 하루(완벽한 24시간)을 투자하고 대략 300번 이상의 구글링을 하고 나니 정상적인 사고가 불가능해졌다.   

구글링을 통해 얻는 자료들은 전부 라즈비안으로만 하거나 데비안만 알려주기 때문에 CentOS를 애용하는 필자에겐 고문과 같았다.

이에 필자는 꼼수를 하나 생각해냈다.

> 라즈비안이던 CentOS던 elf랑 dat파일이 있는데, 크게 다를게 없으니 라즈비안에서 rpi-update를 하고 centos로 붙여넣으면...?

그렇게 1번 과정에서 SSD에 라즈비안을 올리는 대신 CentOS를 올리고 나머지 과정을 같게 해보았다.

~~어림도 없....?~~

오! 예상외의 결과다 CentOS가 부팅된다!

> 드디어? Yes 근데 저장소 용량 왜이래...?

* * *
### 6. 설치는 했는데... 저장소가 심히 아파보인다.

드디어 500GB SSD에 CentOS를 올렸고, SSD로 부팅도 성공했다.   
~~T7을 라즈베리에 못쓸 것 같아 내꺼하려고 했는데 루팡에 실패했다~~

설치가 끝났으니 당연히 저장소부터 확인해야하지 않겠는가?

```linux
root@localhost # df -h
FileSystem  Size Used Available Use%
/dev/root   1.7G 1.2G 	486M	 70
...
```

맞다 사실 df -h를 실행하면 여러가지 항목이 표시되지만 필자는 저 부분 이후로 눈에 들어오지 않았다.   
root를 제외한 모든 공간의 합산이 고작 5GB에 불과했으니 500GB를 달아줬더니 5GB만 쓰고 부족하다고 시위하는 셈이다.

> 필자 격분

fdisk를 사용해서 구조를 다시 잡아보자!

root의 파티션은 sda3였기에.. 파티션 sda3를 날리고   
primary 파티션으로 다시 만들어 주었다.

> 이 과정에서 sda를 sad로 잘못 입력하여 오류가 한 번 났었는데, 그게 내 미래의 감정표현일 줄은 몰랐다.

다시 만들고 resize2fs /dev/sda3를 시전!   
~~ 오류입니다. ~~

> 불길한 기운이 나를 감싸네...

재부팅을 안해서 그런것일거라고 생각 재부팅을 시전했다.

> 안녕 나의 CentOS

* * *
### 7. 잃어버린 나의 Root를 찾아서 ★

사실 Root를 굳이 찾을 필요는 없었다.   
이미 지칠대로 지친 상태였기 때문에 그냥 갈아엎고 재설치를 시전해주었다.

> SSD가 팔팔해졌다

이번엔 파티션을 날리고 재생성하는 짓따위를 할 수는 없었다.   
resize2fs의 경우도 이미 충분한 공간이 할당되어서 할 필요가 없다고 한다.   
~~500GB중에 1.7GB가 충분?~~

CentOS armhfp버전에는 그나마 다행이게도 root를 확장하는 커맨드가 내장되어있었다.

> /usr/bin/rootfs-expand

어쩌면 필자처럼 날려먹는 경우가 많았을지도..

아무튼 해당 명령어를 입력하면 저장공간 내에 남은 공간을 알아서 root 공간으로 확장해주기 때문에   
굳이 추가 설정 없이도 root를 확장할 수 있었다.

이로써 필자의 CentOS설치기가 끝을 볼 수 있었다.

* * *
__NOTE__

처음 필자는 라즈베리파이가 2A짜리 스마트폰 충전기로 충분히 구동이 가능하다고 생각해서 어댑터를 구매하지 않았었는데, 작동이 되지 않았다.   

3A짜리 어댑터가 필요하다는 정보를 입수 배송을 기다리고 있는 과정에서 다시 한 번 시도를 해보았다.

> 놀랍게도 작동이 된다 ^^..

> 모니터에 연결하여 사용하지 않아서 파악이 불가능했던 것으로 밝혀져...

CentOS를 라즈베리4에 설치하기 희망하는 분들은 해당 글을 참고해서 고생을 덜 했으면 좋겠다고 생각해서 만들어진 리포지터리

* * *
>Version English

# RaspberryPI 4B CentOS Setting with SSD
## CentOS7

If you're in hurry, please check ★
The order of ★is 5 -> 1 -> 7
The main point is just execute rpi-eeprom update in your Rasbian, and copy elf and dat file to CentOS.   
So what you really need is Rasbian SDcard and CentOS SSD.

> At this point, only version 7 of CentOS support Raspberry PI   
> Based on 2020-01-11, only the CentOS7 Version 1810 was operational,   
> The version of CentOS7 2003 is also available at the current time of 2020-08-12.

> CentOS7.8.2003 Version   
[CentOS-Userland-7-armv7hl-RaspberryPI-Minimal-4-2003-sda.raw.xz](http://mirror.freethought-internet.co.uk/centos-altarch/7.8.2003/isos/armhfp/CentOS-Userland-7-armv7hl-RaspberryPI-Minimal-4-2003-sda.raw.xz)

Unfortunately, in case of Raspberry4, it cannot use GNOME or KDE version.   
Raspberry4 can only use the version which has number 4 behind Minimal, elses can use on Raspberry 2 or 3   

* * *
## Installing CentOS7

I my case, I tried to install CentOS using two methods, Rufus and Raspberry Pi Imager   

> Link for Rufus   
[Rufus](https://rufus.ie/)

> Link for Raspberry PI Imager   
[Raspberry PI Imager](https://www.raspberrypi.org/downloads/)

* * *
## Attempting to boot from SSD
### Search Information from Google ★

In fact, it was my first time using a raspberry pi, so I knew how to boot with an SD card, but I didn't know how to boot with an SSD.   
In addition, what I bought was the latest version of Raspberry made me even more lost when I heard that the software was unstable.   

This is the data I found while wandering.   
[Official Raspberry Pi 4 USB Boot from SSD (beta)](https://peyanski.com/official-raspberry-pi-4-usb-boot/)   

When you enter this site, you can see very detail explanation how to boot Raspberry PI with your SSD.   
However, unfortunately, it's limited on Rasbian.

Commands

```linux
sudo apt-get update
sudo apt-get upgrade -y
sudo rpi-update
sudo reboot
```

First of all, update all of your package and then reboot the system.   
After rebooting your system, login to ssh to execute this commands

> To install rpi-eeprom we will use this command.   
> In some cases, system may already got this package, in this case, you can skip this step.   
```linux
sudo apt install rpi-eeprom -y
```

After finish installing, enter following commands
```linux
sudo sed -i 's/critical/beta/g' /etc/default/rpi-eeprom-update
```

In case of 2GB or 4GB Model, they have some errors on USB booting menu, so it doesn't have USB booting system.   
As a result of this errors, we need to update our bootloader by using this command. 

> sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/beta/pieeprom-yyyy-MM-dd.bin

yyyy-MM-dd is for the release date of the firmware, the most recent package is 2020-07-31.
Thus, the commands will seems like this.

```linux
sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/beta/pieeprom-2020-07-31.bin
```

After you execute this command, you will get the following results.

> .bin   
> BCM2711 detected    
> Dedicated VL805 EEPROM detected   
> BOOTFS /boot   
> *** INSTALLING /lib/firmware/raspberrypi/bootloader/beta/pieeprom-2020-07-31.bin ***   
> BOOTFS /boot   
> EEPROM update pending. Please reboot to apply the update.   

When you see this results, reboot your system.

To check your bootloader, once more login ssh, and then execute this commands.

```linux
vcgencmd bootloader_version
vcgencmd bootloader_config
```

When you execute these commands, you need to check at least two lines. 

First of all, for the bootloader_version,    
> July 31 2020 --:--:--   
You nned to check the console returns this result.

For the bootloader_config,   
> BOOT_ORDER=0xf41   
The last line of the result should be seems like this.

__0xf41is 4 : USB Booting 1 : SDcard Booting so the system will be boot your RaspberryPi by following this order USB->SDcard

Almost done,   
when you finish following steps, you can boot your rasbian OS with SSD  after execute this commands.

```linux
sudo mkdir /mnt/mydisk
sudo mount /dev/sda1 /mnt/mydisk
sudo cp /boot/*.elf /mnt/mydisk
sudo cp /boot/*.dat /mnt/mydisk
```

Now you can boot your Raspberry with SSD

> When I follow this step, I really can boot my Raspberry with SSD.

* * *
### 2. Follow step 1 with CentOS

> The result was ...

When I start this step, I already saw that the rasbian works well, so as similar as rasbian,   
I think I will get the same result when I use CentOS.   
~~Not a chance~~

Because Centos doesn't have rpi-eeprom package, I can't go furthur aftert doing yum update.
However, fortunately, the bootloader already setted BOOT_ORDER=0xf41,   
So I just copy .elf files and .dat files!

~~Not a chance, do it again~~

> Let's skip this step.

* * *
Updating...
