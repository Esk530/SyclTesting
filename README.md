# SYCL の使い方を練習しながら覚えていく

SYCL の基本理念みたいなものはあまり興味ないので、飛ばしてなんか覚えなきゃいけなさそうなものを確認していく。

# キュー(sycl::queue)

SYCLでは、１つのコードで複数のデバイスを動作させることができる。
そのため、通常の処理では動作だけを記述すればよくても、どのデバイスでそれを実行するかも支持しなければならない。
その司令塔がホストで、プログラムを実行しているPCである。

司令塔が役割と処理を紐づけるのがキューである。
キューはひとまずは、デバイスとの紐づけてして認識しておく。

## デバイスの選択

キューの第一歩はデバイスの選択である。
１つのキューはずっと１つのデバイスと紐づいているべきで、途中で変わることもないだろうからコンストラクタで定義する。
デバイスの選択は以下から

- Default constractor : デフォルト(何がデフォルトなのかは知らん)
	確認したければ、なんか複雑だし、ほかに何が出力できるのかが気になる感じであるが、
	``` cpp
		sycl::queue Q;
		std::cout << Q.get_device().get_info<sycl::info::device::name>() << std::endl;
	```
	という感じでできる。
	
- host_selector : ホスト
	これは、デバッグが主用途だと思う。
	``` cpp
		sycl::queue Q(sycl::host_selector());
	```
	て感じか？

- gpu_selector : 
	GPUで動作させる場合は、これらしい
	``` cpp
		sycl::queue Q(sycl::gpu_selector());
		
こんな感じだが、搭載しているGPUは１つじゃないし、gpu_selectorにもう少し拡張性があるんだと思う。

# テスト

# First test

うまくいかない場合も記録したいので、とりあえず理解の範囲で書いてみる。

今の自分の環境では、CPUが１つとGPUが１つである。
キューを解さないと、デバイス名が取れないということはないだろうが、今わかる方法でデバイス名を出力してみる。

``` cpp
	// Queue Settings
	// 間違い : sycl::queue CPU_Q(sycl::cpu_selector());
	sycl::queue CPU_Q(sycl::cpu_selector{});
	//	sycl::qu eue GPU_Q(sycl::cpu_selector());
	// Output Device name
	std::cout << CPU_Q.get_device().get_info<sycl::info::device::name>() << std::endl;
	//std::cout << GPU_Q.get_device().get_info<sycl::info::device::name>() << std::endl;
```

これで、
Intel ***
Nvidia ***

がでてこい！って感じで

GPUのほうの queueが動かない！
そもそもSYCL が GPUを認識していない気もするな。。。

ひとまずGPUは使わないので、あと回し！

	