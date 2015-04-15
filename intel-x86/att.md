# att常用指令


## lea

lea指令为ATT中用于将寻址得到的地址存储到一个寄存器中的指令


    segreg:base_address(offset_address, index, size)

寻址得到的结果是segreg:base_address+offset_address+index*size

    movl $0x08,%ecx
    lea idt(,%ecx,8),%esi

其完成的事情为像esi中写入一个地址,其值为8(ecx)*8+idt的起始地址,这个就是要向idt表的第8个中写入一个中断描述符的准备工作.
