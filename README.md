# LTC6804-2
ESP32 (esp-idf) driver for battery monitor LTC6804-2

## Example of use:
### Prerequisites
```c
#define POLL_INT			100
#define ERR_CHK(x, m)			do { res = (x); if (x) ERROR(m); \
						} while (0);
```
### Initialization
```c
static spi_device_handle_t *spi;
static void io_send_receive(uint8_t *tx, uint8_t *rx, uint32_t size)
{
	spi_send_receive(spi, tx, rx, size);
}

static void sleep_ms(uint32_t ms)
{
	delay_ms(ms);
}

ltc6804_init_conf conf = {
      .timeout        = TIMEOUT_120_MINUTES,
      .refon          = true,
      .fast_adc       = false,
      .over_voltage   = 4.1,
      .under_voltage  = 3.7,
};

spi = spi_dev(SPI_CS_PIN, 500000, SPI_IDLE_CLOCK_HIGH);

int res = ltc6804_init(&conf, io_send_receive, sleep_ms);
if (res)
      ERROR("LTC6804-2 init failed: %s", strerror(res));
```
### Getting the result
```c
cell_meas cells;
comb_meas comb;
aux_meas aux;
misc_meas misc;
int res;
int nof_cells = 8;

ERR_CHK(ltc6804_convert_cell(&cells, ADC_26Hz_2kHz,
  is_ballancing, POLL_INT), "Cells ADC conversion failed");

ERR_CHK(ltc6804_convert_comb(&comb, ADC_26Hz_2kHz,
  is_ballancing, POLL_INT), "Combined ADC conversion failed");

ERR_CHK(ltc6804_convert_aux(&aux, ADC_26Hz_2kHz, POLL_INT), "AUX ADC conversion failed");

for (int i = 0; i != nof_cells; i++)
  INFO("Cell %u: %.3f V", i, cells.cell[i]);
INFO("Ref: %.3f v", aux.ref_voltage);

INFO("Motor supply voltage: %.2f V", analog_conv(BLDC_VOLTAGE, &aux));
INFO("Motor current cons: %.2f A", analog_conv(BLDC_CURRENT, &aux));
INFO("Battery voltage: %.2f V", analog_conv(BATT_VOLTAGE, &aux));
INFO("Charger voltage: %.2f V", analog_conv(CHRG_VOLTAGE, &aux));
INFO("Charger current: %.2f A", analog_conv(CHRG_CURRENT, &aux));


ERR_CHK(ltc6804_convert_misc(&misc, ADC_26Hz_2kHz, POLL_INT),
  "Reading misc failed");
INFO("Temperature: %.2f C", misc.die_temp);
INFO("Cells summary: %.2f V", misc.sum_meas);
INFO("Analog supply voltage: %.2f V", misc.analog_supp_v);
INFO("Digital supply voltage: %.2f V", misc.digital_supp_v);
```
### Output example
```
I (6286) monitor: Cell 0: 3.281 V
I (6286) monitor: Cell 1: 3.269 V
I (6286) monitor: Cell 2: 3.261 V
I (6286) monitor: Cell 3: 3.271 V
I (6296) monitor: Cell 4: 3.295 V
I (6296) monitor: Cell 5: 3.283 V
I (6306) monitor: Cell 6: 3.260 V
I (6306) monitor: Cell 7: 3.252 V
I (6306) monitor: Ref: 3.000 v
I (6786) monitor: Temperature: 23.29 C
I (6786) monitor: Cells summary: 27.55 V
I (6786) monitor: Analog supply voltage: 5.24 V
I (6786) monitor: Digital supply voltage: 3.12 V
```