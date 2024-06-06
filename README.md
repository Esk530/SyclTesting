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

# メモリ(つまりは配列)の扱い

メモリは、CPU/GPUに独立に存在する。
これらは、別のデバイスなので例えばCPU から GPU のメモリにアクセスしようとすると時間がかかる。
なので、最低限の頻度でコピーする。

SYCLでは、なんかうまくやってくれるっぽいけど、明示的に行うこともできるようである。

## 明示的な方法

handler は、 内部にメンバー関数 memcpy を持っているので、これを使用するとCPU<-> GPUでのメモリのコピーが可能。

``` cpp
	
	std::vector<int> A(100);
	int* dA = malloc_device(100);
	q.submit([&](handler& h)
	{
		h.memcpy(dA,A.data(),100*sizeof(int));
	});
```

みたいな形式でいけるっぽい。


## Buffers を使用する方法

bufferは、メモリを管理するためのオブジェクトで、std::vector と紐づけておけばそれで、ホストとデバイスの両方からアクセス可能なメモリ領域を確保してくれるっぽい。
ただし、これは直接アクセスできないので、 ``accesor`` を媒介する必要がある。
accessorは、第一引数に、buffer をとり、第二引数にはモードをとる。
モードは、
- read_only
- write_only
- read_write
を使用することができる。

accesorは、handler.parallel_forの中でアクセスできる。

問題は、いつホストの　buffer が書き換えられているか？なのよね。
勝手にコピーされるのは、管理が複雑になるため少しいやだなと思ってしまう。


``` cpp

	/**
	* まだ原因はわからないが、gpu デバイスを見つけることはできないので、
	* CPUで動かすことにするとして
	*/

	#include <sycl/sycl.hpp>
	#include <vector>
	#include <iostream>
	#include <string>

	int main()
	{
		const int N = 100;

		// initialize host array
		std::vector<int> A(N, 1),B(N, 2),C(N, 0);
		 
		// create buffer to prepair calculating in device
		sycl::buffer A_buf(A),B_buf(B),C_buf(C);

		// setting queue
		sycl::queue Q;
		std::cout << "Start of device : "
			<< Q.get_device().get_info<sycl::info::device::name>() << std::endl;
		Q.submit([&](sycl::handler& h)
			{
				sycl::accessor a(A_buf, h, sycl::read_only);
				sycl::accessor b(B_buf, h, sycl::read_only);
				sycl::accessor c(C_buf, h, sycl::write_only);
				h.parallel_for(N,[=](sycl::id<1> i) {
					c[i] = a[i] + b[i];
				});
			}
		);
		Q.wait();
		
		/**
		* accesor で c に結びついているとはいえ、Cに
		* そのままいけるのは、少し意味が解らない。
		* 勝手にホストにコピーしているんだとしたら、。。。
		* host ですべて完結しているからなのか？
		*/
		for (int i = 0; i < N; i++)
		{
			std::cout << C[i] << std::endl;
		}
			
	}


```
