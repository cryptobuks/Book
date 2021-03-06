Процесс разработки монеты на основе Litecoin 0.8.7.5
===============================================================

Установка VM
------------

Устанавливаем [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

[Скачиваем образ Ubuntu 16.04.3 х64](https://www.ubuntu.com/download/desktop)


Минимальные системные требования для *физической машины*:

ОЗУ: не менее ? Гб
Жёсткий диск: не менее ? Гб
Процессор: не менее ? ядра (количество ядер физического процессора влияет на объём выделяемых ядер для VM)


Минимальные системные требования для *виртуальной машины*:

ОЗУ: не менее 2 Гб
Жёсткий диск: не менее 20 Гб
Процессор: не менее 1 ядра (количество ядер процессора влияет на скорость компиляции)



https://www.youtube.com/watch?v=mDGxGYvkDEE

Настройка OS и установка зависимостей
-------------------------------------

Обновляем систему:

	sudo apt-get update
	sudo apt-get upgrade

Устанавливаем софт облегчающий работу:

	sudo apt-get install htop mc terminator pydf

Устанавливаем библиотеки разрабоки и гит:

	sudo apt-get install git
	sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils
	sudo apt-get install libboost-all-dev
	sudo apt-get install software-properties-common
	sudo add-apt-repository ppa:bitcoin/bitcoin
	sudo apt-get update
	sudo apt-get install libdb4.8-dev libdb4.8++-dev
	sudo apt-get install libminiupnpc-dev
	sudo apt-get install libzmq3-dev
	sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
	sudo apt-get install libqt4-dev libprotobuf-dev protobuf-compiler

### Клонируем репозиторий с litecoin:

	git clone -b 0.8 https://github.com/litecoin-project/litecoin.git

Редактируем иходный код litecoin:
--------------------------------

### !Необходимо устранить ошибку из предыдущей ветки Litecoin core:

В папке `litecoin/src` в файле `rpcrawtransaction.cpp` на строке `242` изменяем строку как показано ниже:

-`const CScriptID& hash = boost::get<const CScriptID&>(address);`

+`const CScriptID& hash = boost::get<CScriptID>(address);`

### Заменяем все что связано с litecoin на своё:

Переименовываем папку litecoin на название вашей монеты:

	mv litecoin <yourcoin>

Переходим в папку вашей будущей монеты:

	cd <yourcoin>

Далее заменяем всё, что касается litecoin в исходниках, на название вашей монеты:

	find . -type f -print0 | xargs -0 sed -i 's/litecoin/<yourcoin>/g'
	find . -type f -print0 | xargs -0 sed -i 's/Litecoin/<Yourcoin>/g'
	find . -type f -print0 | xargs -0 sed -i 's/LiteCoin/<Yourcoin>/g'
	find . -type f -print0 | xargs -0 sed -i 's/LITECOIN/<YOURCOIN>/g'
	find . -type f -print0 | xargs -0 sed -i 's/LTC/<YOUR>/g'
*Например <yourcoin> = testcoin <YOUR> = TST*

Волшебная команда! Позволит не переименовывать файлы в ручную.

	find * -name '*litecoin*' | sed 'p;s:litecoin:<yourcoin>:g' | xargs -n2 mv

Делаем пробную компиляцию:

	cd src
	make -f makefile.unix

Если компиляция прошла успешно в папке `src` появится файл с названием вашей монеты с окончанием d <yourcoind>. Если компиляция прошла с ошибками - внимательно читаем консольный вывод на предмет ошибок компиляции.

### Замена портов

https://www.youtube.com/watch?v=U2NZaw-c4gE

Далее таким же способом меняем порты на свои: 

	find . -type f -print0 | xargs -0 sed -i 's/9333/<порт>/g'
	find . -type f -print0 | xargs -0 sed -i 's/9332/<порт на 1 меньше>/g'
*Например <порт> = 5988 & <порт на 1 меньше> = 5987*

### Изменяем префиксы в адресах монеты: 
*(вместо расписывания всего процесса здесь должна быть ссылка на документ coin_configuration.md с указанием пункта документа)*

В директории `<yourcoin>/src` в файле `base58.h` на строке 275 изменить следующий код:

-`PUBKEY_ADDRESS = 48, // Litecoin addresses start with L`

+`PUBKEY_ADDRESS = <своё значение>, // Yourcoin addresses start with <свой символ>`
*Для главной сети монеты*

-`PUBKEY_ADDRESS_TEST = 127, // Litecoin test addresses start with l`

+`PUBKEY_ADDRESS_TEST = <свое значение>, // Testcoin test addresses start with <свой символ>`
*Для тестовой сети*

!Для того чтобы правильно подобрать значение соответсвующее символу см. [тут](https://en.bitcoin.it/wiki/List_of_address_prefixes)

### Генерация новых пар ключей:

Переходим в домашнюю директорию:

	cd ~

Для генерации пар ключей вводим следующие команды:

	openssl ecparam -genkey -name secp256k1 -out alertkey.pem
	openssl ec -in alertkey.pem -text > alertkey.hex
	openssl ecparam -genkey -name secp256k1 -out testnetalert.pem
	openssl ec -in testnetalert.pem -text > testnetalert.hex
	openssl ecparam -genkey -name secp256k1 -out genesiscoinbase.pem
	openssl ec -in testnetalert.pem -text > genesiscoinbase.hex

Далее выполняем команду:
	
	cat alertkey.hex

В консоли появится следующее:
	
		Private-Key: (256 bit)
	priv:
	    0b:83:a8:03:b0:76:4c:8e:df:3e:cc:87:67:31:c0:
	    a1:94:ae:c9:97:8c:22:b3:27:24:cc:74:ef:01:95:
	    ff:c3
	pub: 
	    04:57:4c:42:95:11:c9:88:4a:f9:63:d4:3c:91:8e:
	    0a:71:ca:a3:64:cc:d6:98:da:b8:73:b2:5f:05:89:
	    21:1f:52:c7:b8:fd:1d:69:ea:2a:21:7d:5d:85:21:
	    6a:4f:7a:18:9b:e5:85:0d:ac:4d:f5:9f:ec:03:a5:
	    32:1a:e2:a0:84
	ASN1 OID: secp256k1
	-----BEGIN EC PRIVATE KEY-----
	MHQCAQEEIAuDqAOwdkyO3z7Mh2cxwKGUrsmXjCKzJyTMdO8Blf/DoAcGBSuBBAAK
	oUQDQgAEV0xClRHJiEr5Y9Q8kY4KccqjZMzWmNq4c7JfBYkhH1LHuP0daeoqIX1d
	hSFqT3oYm+WFDaxN9Z/sA6UyGuKghA==
	-----END EC PRIVATE KEY-----

Далее в папке <yourcoin>/src находим файл alert.cpp и переходим к строке 22 и заменяем текущий pub-ключ:

	pub: 
	    04:57:4c:42:95:11:c9:88:4a:f9:63:d4:3c:91:8e:
	    0a:71:ca:a3:64:cc:d6:98:da:b8:73:b2:5f:05:89:
	    21:1f:52:c7:b8:fd:1d:69:ea:2a:21:7d:5d:85:21:
	    6a:4f:7a:18:9b:e5:85:0d:ac:4d:f5:9f:ec:03:a5:
	    32:1a:e2:a0:84

на ключ, который видим в консоли:
	
-`static const char* pszMainKey = "04f0801cbbf15f959de9e345888efb74222fa13844d5a428e33c52c0dbb8cb9c1a654bde53bd2415c4d7f54ba9b92beabc7c3074d5c1bd91b971072fec7cdfcd94";`

+`static const char* pszMainKey = "04:57:4c:42:95:11:c9:88:4a:f9:63:d4:3c:91:8e:0a:71:ca:a3:64:cc:d6:98:da:b8:73:b2:5f:05:89:21:1f:52:c7:b8:fd:1d:69:ea:2a:21:7d:5d:85:21:6a:4f:7a:18:9b:e5:85:0d:ac:4d:f5:9f:ec:03:a5:32:1a:e2:a0:84";`

Затем удаляем все двоеточия внутри ключа.

В домашней директори выполняем команду:

	cat testnetalert.hex

После этого в файле alert.cpp для строки 23 повторяем действия описаные со строки 147 по 162

-`static const char* pszTestKey = "04d429c63946b6f404b387c0ef6a49790d3c904e80004afc223b306b148ed52e2dbd412fc88d121c16ad9658a6a165300c14ac3cf8d111929c8123c93c5022cb87";`

+`static const char* pszTestKey = "Другой ключ";`

Узнать для чего необходимы эти ключи можно [здесь](https://bitcoin.stackexchange.com/questions/583/what-is-the-alert-system-in-the-bitcoin-protocol-how-does-it-work)

В папке <yourcoin>/src открываем файл main.cpp переходим к строке 2788

В консоли пишем:

	cat genesiscoinbase.hex

Заменяем текущий ключ на pubkey из файла genesiscoinbase.hex как мы это делали выше.

-`txNew.vout[0].scriptPubKey = CScript() << ParseHex("04d429c63946b6f404b387c0ef6a49790d3c904e80004afc223b306b148ed52e2dbd412fc88d121c16ad9658a6a165300c14ac3cf8d111929c8123c93c5022cb87") << OP_CHECKSIG;`

+`txNew.vout[0].scriptPubKey = CScript() << ParseHex("<pub-ключ из файла genesiscoinbase.hex>") << OP_CHECKSIG;`

### Магические биты 
*(вместо расписывания всего процесса здесь должна быть ссылка на документ coin_configuration.md с указанием пункта документа)*

Открываем файл `main.cpp` и переходим к строке `2745` изменям шестнадцатиричные значения на любые свои

	pchMessageStart[0] = 0xf8;
	pchMessageStart[1] = 0xc4;
	pchMessageStart[2] = 0xbf;
	pchMessageStart[3] = 0xd8;

Эти биты будут вставлятся впереди сообщения для того чтобы игнорировать сообщения от сторонних программ если они обращаются к порту, который занят нашей монетой.

Далее в этом же файле переходим к строке 3082 и также прописываем свои биты:

	unsigned char pchMessageStart[4] = { 0xf2, 0xca, 0xbf, 0xd2 }; // Yourcoin: absolute random.

### Удаление DNS Seed
*(вместо расписывания всего процесса здесь должна быть ссылка на документ coin_configuration.md с указанием пункта документа)*

Открываем файл `net.cpp` в папке src и переходим к стоке 1175 и удалем все кроме значения `{NULL, NULL}`

	static const char *strMainNetDNSSeed[][2] = {
	    {NULL, NULL}
	};

Тоже самое делаем для константы `strTestNetDNSSeed`

	static const char *strTestNetDNSSeed[][2] = {
    {NULL, NULL}
	};

### Удаление захардкоженых Node
*(вместо расписывания всего процесса здесь должна быть ссылка на документ coin_configuration.md с указанием пункта документа)*

В файле `net.cpp` переходим к строке `1226`, здесь необходимо удалить все значения и прописать `0x0`

	unsigned int pnSeed[] =
	{
	    0x0
	};

### Настройка основных параметров монеты

Настройка параметров монеты производились в операционной системе Ubuntu. Далее будут представлены строки которые были изменены нами (*изменяемая* часть заключена в <>), номера строк могут не совпадать в вашем случае. **Все файлы, кроме `debug.log` (который нужно закрывать), следует держать открытыми!**

`директория монеты/src/` файл `main.cpp` строка  `1090`

	int64 nSubsidy = <50> * COIN //количество монет за генерируемый блок
	nSybsidy>>=(nHeight/<840000>) // уменьшение награды за н-кол-во блоков
	static const int64 nTargetTimespan= 3.5 * 24 * 60 * 60 //время, через которое будет происходить корректировка сложности
	static const int64 nTargetSpacing = 2.5 * 60 //время, генерации блока

Файл `main.h`

	static const int64 MAX_MONEY =  <84000000> * COIN //максимальное количество монет
	static const int64 COIN_MATURITY = <100> // количество блоко для подтверждения транзакции

строка  627

	return dPriority > COIN * <720> / 250 //720 - количество блоков, максимально генерируемых за один день

### Изменение генезис блока

файл `main.cpp` строка ~ `2787` нашем случае

	txNew.vout[0].nValue = <88> * COIN; //количество монет в генезис блоке
	const char* pszTimestamp = "<NY Times 26/Jan/2018 The U.S. May Back a Vietnam Coal Plant. Russia Is Already Helping.>"; //нужно изменить комментарий

*Изменив данную строчку и скомпилировав кошелек заново вы уже не сможете участвовать в действующей сети. C этого момента начинается создание собственной сети монеты (т.н. hardfork).
Используя свежую новость при создании собственного Genesis block’а вы получаете неопровержимое доказательство о дате начала работы сети.*

строка `2800` 
	
	block.nTime    = <1516978813>;//сообщает количество секунд с 1 января 1940 года, данная строка заключена в блок "if" т.е. это значение для тестовой сети

редактирование производилось из под убунты. для того чтобы сгенерировать это число в терминале убунты пишем:

	data +%s

строка `2794`
	
	block.nTime  = <1516978950>;//аналогично строке выше. повторяем описанные выше действия.

строка ~ `2851`

-`assert(block.hashMerkleRoot == uint256("<bc01ce873e0100fdfb30a5e817296f8b0d6a2b9f0da3cd8e47ed47b0dc0313ae>"));//необходимо изменить на`

+`assert(block.hashMerkleRoot == uint256("0x"));//сохраняем файл, не закрываем`

Открываем терминал, переходим в директорию монеты `yourcoin/src/`

	cd yourcoin/src/
	make -f makefile.unix
	./yourcoind 

консоль не закрываем!

Открываем проводник, переходим в домашний каталог, используем комбинацию клавишь Ctrl+H, каталог `.yourcoin`, файл `debug.log` (home/.yourcoin/debug.log)

Копируем последнюю строку, возвращаемся в файл `main.cpp` и изменяем строку:

	assert(block.hashMerkleRoot == uint256("<>")ur

Копируем следующий блок кода: *блок кода отвечает за генерацию генезис блока*

	if (true && block.GetHash() != hashGenesisBlock)
			{
				printf("Searching for genesis block...\n");
				// This will figure out a valid hash and Nonce if you're
				// creating a different genesis block:
				uint256 hashTarget = CBigNum().SetCompact(block.nBits).getuint256();
				uint256 thash;
				char scratchpad[SCRYPT_SCRATCHPAD_SIZE];
	 
				loop
				{
	#if defined(USE_SSE2)
					// Detection would work, but in cases where we KNOW it always has SSE2,
					// it is faster to use directly than to use a function pointer or conditional.
	#if defined(_M_X64) || defined(__x86_64__) || defined(_M_AMD64) || (defined(MAC_OSX) && defined(__i386__))
					// Always SSE2: x86_64 or Intel MacOS X
					scrypt_1024_1_1_256_sp_sse2(BEGIN(block.nVersion), BEGIN(thash), scratchpad);
	#else
					// Detect SSE2: 32bit x86 Linux or Windows
					scrypt_1024_1_1_256_sp(BEGIN(block.nVersion), BEGIN(thash), scratchpad);
	#endif
	#else
					// Generic scrypt
					scrypt_1024_1_1_256_sp_generic(BEGIN(block.nVersion), BEGIN(thash), scratchpad);
	#endif
					if (thash <= hashTarget)
						break;
					if ((block.nNonce & 0xFFF) == 0)
					{
						printf("nonce %08X: hash = %s (target = %s)\n", block.nNonce, thash.ToString().c_str(), hashTarget.ToString().c_str());
					}
					++block.nNonce;
					if (block.nNonce == 0)
					{
						printf("NONCE WRAPPED, incrementing time\n");
						++block.nTime;
					}
				}
				printf("block.nTime = %u \n", block.nTime);
				printf("block.nNonce = %u \n", block.nNonce);
				printf("block.GetHash = %s\n", block.GetHash().ToString().c_str());
			}
			

Вставляем его после блока(~ 2803 строка):

	if (fTestNet)
    {
        block.nTime    = 1516978813;
		block.nNonce   = 389064586;
    }
	
строка `38`

-`uint256 hashGenesisBlock("0xb5070bb4044bade3bd85a0ab1dc76499492881e195de25d41d6e5c88827db57c");` 

+`uint256 hashGenesisBlock("0x");`

строка `2749`

-`hashGenesisBlock = uint256("0xf1ad49a505cd63a53d06f5961d9d63e7575743525bfad90d5b8770bdcaa55bac");`

+`hashGenesisBlock = uint256("0x");`

Возвращаемся в терминал выполняем компиляцию и запускаем демона для тестнета.

	make -f makefile.unix
	./<yourcoin>d -testnet

Ждем появления сообщения об ошибке

Открываем проводник, открываем `debug.log` тестнета (home/.yourcoin/testnet3/debug.log)
переходим к строке

	block.nNonce = ...

копируем выражение после знака "="

Возвращаемся в `main.cpp` и изменяем строку `2801` ( block.nNonce   = <...>;)

	block.nNonce   = <вставляем скопированное выражение>;

Возвращаемся в `debug.log` тестнета (home/.yourcoin/testnet3/debug.log).
Находим строку

	block.GetHash = ... //многоточие - некоторое буквенноцифровое выражение

копируем выражение после знака "="

Возвращаемся в `main.cpp` и изменяем строку `2749` (hashGenesisBlock = uint256("0x"); изменение которой производилось выше)

	hashGenesisBlock = uint256("0x<вставляем скопированное выражение>");

Возвращаемся в терминал выполняем компиляцию и запуск демона уже для mainnet.

	make -f makefile.unix
	./<yourcoin>d

Открываем debug.log (home/.yourcoin/debug.log) и переходим к строке

	block.nNonce = ...

копируем выражение после знака "="

Возвращаемся в `main.cpp` и изменяем строку `2796` ( block.nNonce   = <...>;)

	block.nNonce   = <вставляем скопированное выражение>;

строка 2803

-`if (true && block.GetHash() != hashGenesisBlock)`

+`if (false && block.GetHash() != hashGenesisBlock)`

Открываем ранее используемый `debug.log` (home/.yourcoin/debug.log) и переходим к строке

	block.GetHash = ...

копируем выражение после знака "="  

Возвращаемся в `main.cpp`, строка `38`

	uint256 hashGenesisBlock("0x");

меняем на

	uint256 hashGenesisBlock("0x<вставляем скопированное выражение>");

Сохраняем *и вообще повнимательнее с сохранениями*

### Удаление чекпоинтов
*(вместо расписывания всего процесса здесь должна быть ссылка на документ coin_configuration.md с указанием пункта документа)*

Открываем проводник, переходим к "директория монеты"/src/ открываем файл checkpoints.cpp
переходим к следующему блоку кода:

	static MapCheckpoints mapCheckpoints =
        boost::assign::map_list_of
        (  1500, uint256("0xc82444ae55ddda29fddd3334a786458aaf47af609b68c65b859df642423f7068"))//3-я строка
		(  4032, uint256("0xc4564afsljghdfgdf9fdgfd5g4a786458aa453cvbxcb68c65b859df642xxfgd4"))//строки ниже включая эту удаляем
		(  8064, uint256("0xc54742e5fgdfgsddd3fdgad4gds6464aaf47af609bdfgsdfg59df64dfgsdf087"))
        ...
		; // удаляем строки до этой строки!
    static const CCheckpointData data = {

в 3 строке блока, 1500 меняем на 0, вместо буквенноцифрового выражения вставляем скопированное ранее
приводим блок кода, приведенный выше, к следующему виду

	static MapCheckpoints mapCheckpoints =
        boost::assign::map_list_of
        (  0, uint256("0x<вставляем скопированное выражение>"))
		; 
    static const CCheckpointData data = {

Возвращаемся в `main.cpp` строка `2794`

	block.nTime    = 1516978950; // копируем выражение после "="

Возвращаемся в файл checkpoints.cpp
переходим к следующему блоку кода:

	static const CCheckpointData data = {
        &mapCheckpoints,
		1516978813, // * UNIX timestamp of last checkpoint block
        4564636,    // * total number of transactions between genesis and last checkpoint
                    //   (the tx=... number in the SetBestChain debug.log lines)
        7000.0     // * estimated number of transactions per day after checkpoint
	};
	
приводим блок кода, приведенный выше, к следующему виду

	static const CCheckpointData data = {
        &mapCheckpoints,
		<Вставляем скопированное значение>, // * UNIX timestamp of last checkpoint block
        <меняем на 0>,    // * total number of transactions between genesis and last checkpoint
                    //   (the tx=... number in the SetBestChain debug.log lines)
        <меняем на 1.0>     // * число транзакций за день после чекпоинта
	};
	
Переходим в `main.cpp`, строка `2798` блок кода

	if (fTestNet)
        {
            block.nTime    = 1516978813; копируем значение после "="
            block.nNonce   = 389064586;
        }

Возвращаемся в файл `checkpoints.cpp`
переходим к следующему блоку кода:

	static MapCheckpoints mapCheckpointsTestnet =
        boost::assign::map_list_of
        (   546, uint256("0xebba27f6345f34cf1c3239c31c196ab165210c13d4712154364c6b76ba5b3b84")) //546 заменяем на 0
        ;
	static const CCheckpointData dataTestnet = {
        &mapCheckpointsTestnet,
        1516978950,		//вставляем скопированное значение
        547,			//заменяем на 0
        576 			//заменяем на 1.0
    };

Сохраняем

### Компиляция кошелька с GUI для Linux

Переходим в терминал

	cd ~/<yourcoin>
	qmake
	make

Смотрим что появилось в текущей директории

	ls
	//отобразится содержимое директории в которой вы находитесь, ищем <yourcoin>-qt !!!!

Делаем пробный запуск кошелька *для тестовой сети с флагом `-testnet`, для основной сети без флага.*

	./testcoin-qt -testnet
	./testcoin-qt

*Чуть радуемся...*

Майнинг первых монет
--------------------

### Подготовка мастер ноды

Для начала необходимо настроить виртуальную машину. В VirtualBox переходим в настройку сети.
В типе подключения указываем *сетевой мост*, в имени указываем физический адаптер, который используеться в настоящий момент.

Далее необходимо склонировать машину с изменением MAC адресов. Для виртуального жесткого диска можно использовать связное клонирование.

После этого запускаем склонированную машину (она же мастер нода), и идем в папку `.название_нашей_монеты`, которая находится в домашней директории. И полностью очищаем ее. Тут же создаем файл `название_нашей_монеты.conf` В созданный файл вставляем:

	server=1
	rpcuser=user
	rpcpassword=password

*Данные параметры пишем только для теста!*

Далее в основной машине переходим в `.название_нашей_монеты`, которая также находится в домашней директории. И также создаем файл `название_нашей_монеты.conf` и вставляем туда:

	addnode=<ip склонированной машины>

*Для того чтобы узнать <ip склонированной машины>*

Теперь на обоих машинах запускаем кошельки, для этого переходим в папку с нашей монетой и находим файл `нашкоин-qt`

И наконец на исходной машине в кошельке в разделе помощь открываем окно отладки и переходим на вкладку консоль. И пишем следущую команду:

	setgenerate true

Теперь ждем добычи первого блока в вашей новой цепи
