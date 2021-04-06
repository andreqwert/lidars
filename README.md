# lidars
ZCM installation instructions


1. Устанавливаем пакеты:
```
sudo apt-get install cython3 python3-dev python3-pip python-dev cython openjdk-8-jre openjdk-8-jdk libelf1 libelf-dev npm nodejs gcc-5 g++-5
```

2. Ставим ZeroMQ:
```
cd <your workspace directory>
wget https://github.com/zeromq/zeromq4-1/releases/download/v4.1.6/zeromq-4.1.6.tar.gz --no-check-certificate
tar -xvzf zeromq-4.1.6.tar.gz
cd zeromq-4.1.6/
./configure
make -j6 # Instead of "6" you should set your number of CPU cores 
sudo make install
```

2.5 (При необходимости дописываем):
```
# example: arm64
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-arm64
```

3. Устанавливаем ZeroCM:
```
git clone https://github.com/ZeroCM/zcm
cd zcm
sudo su
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
CXX=/usr/bin/g++-5 CC=/usr/bin/gcc-5 ./waf configure --use-all --use-third-party
CXX=/usr/bin/g++-5 CC=/usr/bin/gcc-5 ./waf build
CXX=/usr/bin/g++-5 CC=/usr/bin/gcc-5 ./waf install
exit
```

4. Записываем в `zcm/zcm/python` файл `setup.py`, меняем version на 1.6.1 (на 1 апреля issues в github'е советует прописать так).

5. Устанавливаем через pip:
```
cd ..
pip3 install zcm/python
pip3 install setuptools
```

Проверить работоспособность:
```
import zerocm as zcm
```

Чтобы пользоваться удобными штуками типа zcm-spy-lite - продолжаем.     
6. Скачать jdk с https://www.oracle.com/java/technologies/javase-downloads.html
Если скачано в формате `.rpm` для Ubuntu, то надо конвертировать `.rpm` в `.deb`:
```
sudo apt-get install alien
sudo alien jdk-16_linux-aarch64_bin.rpm
```
7. Получится `.deb`, его можно просто открыть через Ubuntu Software и поставить как обычное приложение.    
8. Далее проверяем пути (тут записываем сразу в environment, чтоб каждый раз это не прописывать при выходе из системы):
```
sudo nano /etc/environment
JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64/"
source /etc/environment
Нужно выйти и зайти в систему, затем можно проверить, что всё ок командой:
echo $JAVA_HOME
(На всякий случай) установим еще pyembed:
pip3 install pyembed
```
9. Скачиваем интерпретатор ЯП julia, причём обязательно либо версию 0.6.4, либо 1.3.1 отсюда:
https://julialang.org/downloads/oldreleases/
Скачиваем `.tar.gz`, далее:
```
tar xvzf julia-1.3.1-linux-x86_64.tar.gz
export PATH="$PATH:/path/to/<Julia directory>/bin"
```
Проверить работоспособность можно командой: julia
Ставим символическую ссылку на julia: `sudo ln -s /путь/до/tar/gz/bin/julia /usr/bin/julia`

10. Собираем всё [наконец-то] командой:    
```
sudo ./waf configure --use-all
```
Должно собраться.

11. Устанавливаем дополнительные скрипты (может быть, это необязательно, но я не знаю, так что тоже напишу):
```
./scripts/install-deps.sh
```

12. Далее билдим examples. Если следовать инструкции на гитхабе, то ничего не заработает. Хотя авторы указывают, что для билдинга examples требуется опция `--use-all`, но в следующих же строчках этот `--use-all` не пишут. Поэтому:
```
source ./examples/env
sudo ./waf configure --use-all
./waf build_examples
./build/examples/examples/cpp/sub
```
Запустится поток.

13. Параллельно в соседнем окне терминала вводим путь до библиотеки от zcm_types:
`export ZCM_SPY_LITE_PATH='/home/user/zcm/zcm_types/libzcmtypes.so'`
(Но чтобы не вводить это каждый раз при запуске zcm, лучше добавить это в ~/.bashrc)
Итого запуск можно произвести через:
```
zcm-spy-lite --zcm-url ipc
zcm-logplayer --zcm-url ipc 534_train_2011161419.zcm.1.dl
```

14. [Опционально]
Чтобы в `zcm_types` были скрипты от всех языков (а то у меня вот в lidar_livox ничего, кроме `.zcm`, не лежало), надо эти скрипты сбилдить:
```
cd zcm_types
sudo sh zcm_gen.sh
```
