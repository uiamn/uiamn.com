---
title: Axis interfaceをSDK上で使用する
date: "2020-06-02"
description: "Vivado HLSで引数をaxis interfaceで合成したIPに，SDKから入力する方法"
---

Vivado HLSのinterfaceには様々な物が有るが，その中の引数をaxis interfaceで合成したIPを作成後，それをSDK上で扱う方法が検索しても中々出てこなかったのでメモしておく．

サンプルとして次の様なIPを用意した．

```C
int axis_test(int inp[4], int outp[4]) {
#pragma HLS INTERFACE axis register both port=inp
#pragma HLS INTERFACE s_axilite port=outp
#pragma HLS INTERFACE s_axilite port=return
	for(int i=0; i<4; i++) outp[i] = inp[i]*2;
	return 0;
}
```

`inp[4]`の全ての要素を2倍して`outp[4]`に格納するだけの単純なIP．ここで，`outp`と返り値のINTERFACEはs_axiliteとし，`inp`はaxisとして合成した．合成したらVivadoを開き，PSと接続する．



SDK側のコードは以下．

```C
#include <stdio.h>
#include "platform.h"
#include "xil_printf.h"
#include "xaxis_test.h"
#include "xllfifo.h"

int main() {
    init_platform();
    XAxis_test inst;
    XAxis_test_Initialize(&inst, XPAR_AXIS_TEST_0_DEVICE_ID);

    int data[4] = {0, 1, 2, 3};
    int out[4];

    XLlFifo fifoinst;

    XAxis_test_Start(&inst);
    XLlFifo_Initialize(&fifoinst, XPAR_AXI_FIFO_MM_S_BASEADDR);

    XLlFifo_Write(&fifoinst, data, 4*4);
    XLlFifo_TxSetLen(&fifoinst, 4*4);

    while(XAxis_test_IsDone(&inst) == 0);
    XAxis_test_Read_outp_Words(&inst, 0, out, 4);

    for(int i=0; i<4; i++) printf("%d", out[i]);
    printf("\n");

    cleanup_platform();
    return 0;
}
```



ポイントは`XLlFifo_Write`と`XLlFifo_TxSetLen`の順序．[xllfifo.h](https://github.com/Xilinx/embeddedsw/blob/master/XilinxProcessorIPLib/drivers/llfifo/src/xllfifo.h)には次の様な記述が有る．

>XLlFifo_TxSetLen begins a hardware transfer of _Bytes_ bytes out of the transmit channel of the FIFO specified by _InstancePtr_.

>XLlFifo_Write writes _Bytes_ bytes of the block of memory, referenced by _BufPtr_, to the transmit channel of the FIFO referenced by _InstancePtr_.

Writeでtransmit channelにデータを書き込んで，TxSetLenでデータの転送を開始するらしい．これを逆にするとデータの転送がなされなくなるため，ハードウェア側の処理がいつまでも終了しない．結果， `while(XAxis_test_IsDone(&inst) == 0);`で無限ループに陥る．
