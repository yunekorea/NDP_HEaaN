## How to add new custom driver?

이 문서는 새로운 사용자 정의 드라이버 기능을 추가하는 방법과, 이 기능을 Host 서버에서 호출하는 방법에 대해 설명합니다.

#### Method of adding new driver feature to spdk

1. opcode 등록

    새로운 사용자 정의 드라이버 기능을 구성하기 위해, 예약되지 않은 opcode를 찾아야 합니다. 
    
    nvme 공식 스펙 문서를 읽고, 새로운 opcode를 생각한 뒤 이를 spdk_nvme_nvm_opcode(spdk/include/spdk/nvme_spec.h)에 등록하세요.(ex. 0xd0)
    
    ```c
    enum spdk_nvme_nvm_opcode {
    ...
    SPDK_NVME_OPC_CUSTOM_ECHO = 0xd0, // opcode for custom echo,
    ```

2. 새로운 opcode가 io 명령으로 동작할 수 있도록 구성

    ```c
    static const struct spdk_nvme_cmds_and_effect_log_page g_cmds_and_effect_log_page = {
        ...
        .io_cmds_supported = {
            ...
            /* CUSTOM ECHO */
            [SPDK_NVME_OPC_CUSTOM_ECHO]     = {1, 1, 0, 0, 0, 0, 0, 0},
            ...
    ```

3. 새로운 opcode에 대한 드라이버 함수 작성
    
    파일 경로 `spdk/lib/nvmf/ctrlr_bdev.c` 에 새로운 드라이버 함수를 정의하고 구현합니다.
    
    ```c
    int
    nvmf_bdev_ctrlr_custom_echo_cmd(struct spdk_bdev *bdev, struct spdk_bdev_desc *desc,
                                    struct spdk_io_channel *ch, struct spdk_nvmf_request *req)
    {
     ...
     // 새로운 드라이버 기능 정의
     ...
    ```

4. 새로운 드라이버 기능과 opcode 매핑

    nvmf_ctrlr_process_io_cmd(spdk/lib/nvmf/ctrlr.c) 에 새로운 드라이버 함수를 정의하고 구현합니다.
    
    
    ```c
    int
    nvmf_ctrlr_process_io_cmd(struct spdk_nvmf_request *req)
    {
        ...
        switch (cmd->opc) {
        ...
        // CUSTOM COMMAND
        case SPDK_NVME_OPC_CUSTOM_ECHO:
            return nvmf_bdev_ctrlr_custom_echo_cmd(bdev, desc, ch, req);
    ```

5. Unit Test 등록(중요)

    새롭게 구현한 드라이버 함수에 대해 실제 unit test를 작성하는 것은 아니지만, 구현한 함수를 unit test 영역에 등록하지 않으면 빌드가 실패합니다. 
    따라서 구현한 함수를 반드시 unit test에 등록합니다.
    
    등록 위치: `spdk/test/unit/lib/nvmf/ctrlr.c` , `spdk/test/unit/lib/nvmf/tcp.c`
    
    ```c
    ... 
    DEFINE_STUB(nvmf_bdev_ctrlr_custom_echo_cmd, // 각 파일에 새롭게 작성한 함수에 대해 아래와 같이 추가
            int,
            (struct spdk_bdev *bdev, struct spdk_bdev_desc *desc, struct spdk_io_channel *ch,
             struct spdk_nvmf_request *req),
            0);
    ...
    ```

6. SPDK 빌드 및 재시작

    ```shell
   sudo make -j `nproc`
   sudo build/bin/nvmf_tgt
   ```

#### Method of call new driver feature from host

이전 섹션과 같이 spdk 내에 새로운 드라이버 기능을 정의하면 이를 호스트에서 호출해야 합니다.

1. nvme-cli 및 io-passthru 이용
    `nvme-cli/nvme.c`의 `static int passthru()` 함수를 기호에 맞게 수정하고, 아래와 같이 io-passthru를 호출합니다.
    
    ```shell
    sudo nvme io-passthru /dev/nvme0n1 \
        --opcode=0xd0 \
        --namespace-id=1 \
        --data-len=8192 \
        --cdw10=0 \
        --cdw11=1 \
        --cdw12=2 \
        --cdw13=1 \
        --read \
    ```
   
    위 명령어는 참고용이며, 새롭게 구현한 함수에 맞는 명령어를 직접 구현하시면 됩니다.