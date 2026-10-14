
This directory is intended for PIO Unit Testing and project tests.

Unit Testing is a software testing method by which individual units of
source code, sets of one or more MCU program modules together with associated
control data, usage procedures, and operating procedures, are tested to
determine whether they are fit for use. Unit testing finds problems early
in the development cycle.

More information about PIO Unit Testing:
- https://docs.platformio.org/page/plus/unit-testing.html

# Procedimento de testes para a placa

Iremos aqui elaborar quais testes estão sendo realizados dentro do sistema para assegurar o pleno funcionamento da placa.

## 1. Verificação da existência do Alias dentro da devicetree

Nas linhas 13-24, é feita uma verificação da existência das variáveis dentro do mapa de entradas e saídas do dispositivo em tempo de compilação. Isto é: se a variável estática declarada não for encontrada, o programa sequer executa. Veja:

```
// Verifica se o LED está definido no Device Tree
#if DT_NODE_HAS_STATUS(LED0_NODE, okay)
static const struct gpio_dt_spec led = GPIO_DT_SPEC_GET(LED0_NODE, gpios);
#else
#error "Unsupported board: led0 devicetree alias is not defined"
#endif

#if DT_NODE_HAS_STATUS(LED1_NODE, okay)
static const struct gpio_dt_spec led_1 = GPIO_DT_SPEC_GET(LED1_NODE, gpios);
#else
#error "Unsupported board: led0 devicetree alias is not defined"
#endif
```

## 2. Verificação do estado dos dispositivos
Ao início da execução do programa, o software verifica se o led de entrada está disponível para uso após sua compilação. Com isso é possível verificar se houve a mudança de estado nas especificações do dispositivo dentro da compilação.

```
// Verifica se o device está pronto
    if (!gpio_is_ready_dt(&led)) {
        printk("Error: LED device %s is not ready\n", led.port->name);
        return;
    }
	// Verifica se o device está pronto
    if (!gpio_is_ready_dt(&led_1)) {
        printk("Error: LED device %s is not ready\n", led_1.port->name);
        return;
    }
```

Caso haja retorno negativo, o microcontrolador envia uma mensagem de erro no terminal, apontando o problema no sistema.

## 3. Verificação de mudança de estado
Após a verificação da disponibilidade das entradas do led, é feita a configuração do pino como saída em nível baixo (LOW). essa função retorna erro como valor negativo caso não tenha sido possível fazer a configuração do sistema. Nesse caso, é encaminhado um erro de acordo com um dos possíveis casos:
* -EIO
* -EACCES
* -EBUSY
* -EINVAL
* -ENOSYS ou
* -ENOTSUP

```

    // Configura o pino como saída
    ret = gpio_pin_configure_dt(&led, GPIO_OUTPUT_INACTIVE);
    if (ret < 0) {
        printk("Error %d: failed to configure LED pin\n", ret);
        return;
    }
	
	// Configura o pino como saída
    ret2 = gpio_pin_configure_dt(&led_1, GPIO_OUTPUT_INACTIVE);
    if (ret2 < 0) {
        printk("Error %d: failed to configure LED pin\n", ret2);
        return;
    }

    printk("LED blinking on %s pin %d\n", led.port->name, led.pin);
	printk("LED blinking on %s pin %d\n", led_1.port->name, led_1.pin);
```
