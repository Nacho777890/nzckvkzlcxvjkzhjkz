#include <stdio.h>
#include "pico/stdlib.h"

typedef struct {
  uint8_t rs;	
  uint8_t en;		
  uint8_t d4;	
  uint8_t d5;		
  uint8_t d6;		
  uint8_t d7;		
} lcd_t;


static inline lcd_t lcd_get_default_config(void) {
  lcd_t lcd = { 0, 1, 2, 3, 4, 5 };
  return lcd;
}

static inline uint32_t lcd_get_data_mask(lcd_t lcd) {
  return 1 << lcd.d4 | 1 << lcd.d5 | 1 << lcd.d6 | 1 << lcd.d7;
}

static inline uint32_t lcd_get_mask(lcd_t lcd) {
  return 1 << lcd.rs | 1 << lcd.en | lcd_get_data_mask(lcd);
}

void lcd_init(lcd_t lcd);
void lcd_put_nibble(lcd_t lcd, uint8_t nibble);
void lcd_put_command(lcd_t lcd, uint8_t cmd);
void lcd_putc(lcd_t lcd, char c);
void lcd_puts(lcd_t lcd, const char* str);
void lcd_clear(lcd_t lcd);
void lcd_go_to_xy(lcd_t lcd, uint8_t x,  uint8_t y);


int main() {

  stdio_init_all();
  int i[16];
  char str[]="hola mundo!"
  for(int i=0;i<16;i++)
  printf("%c\n",str[i]);
}
  lcd_t lcd = lcd_get_default_config();
	
	lcd_init(lcd);

  while (true) {
	
    lcd_clear(lcd);
	
		lcd_puts(lcd, "Hello!");
	
		lcd_go_to_xy(lcd, 6, 1);
	
    lcd_puts(lcd, "From Pico!");
  
		sleep_ms(500);
  }
}


void lcd_init(lcd_t lcd) {
  
  uint32_t mask = lcd_get_mask(lcd);

  gpio_init_mask(mask);

  gpio_set_dir_out_masked(mask);

	lcd_put_command(lcd, 0x03);

	sleep_ms(5);

	lcd_put_command(lcd, 0x03);

	sleep_us(100);

	lcd_put_command(lcd, 0x03);

	lcd_put_command(lcd, 0x02);

	lcd_put_command(lcd, 0x02);

  lcd_put_command(lcd, 0x08 | (false << 2));

  lcd_put_command(lcd, 0x00);
  lcd_put_command(lcd, (3 << 2) | (false) | false);

  lcd_put_command(lcd, 0x00);
  lcd_put_command(lcd, 0x06);
}


void lcd_put(lcd_t lcd, uint8_t nibble) {

  uint32_t mask = lcd_get_data_mask(lcd);

	uint32_t value = 	((nibble & 0x8)? true : false) << lcd.d7 | 
										((nibble & 0x4)? true : false) << lcd.d6 |
										((nibble & 0x2)? true : false) << lcd.d5 | 
										((nibble & 0x1)? true : false) << lcd.d4;
 
  gpio_put_masked(mask, value);

	gpio_put(lcd.en, true);

	sleep_us(40);

	gpio_put(lcd.en, false);
}


void lcd_put_command(lcd_t lcd, uint8_t cmd) {

	gpio_put(lcd.rs, false);

	lcd_put(lcd, cmd);
}

void lcd_putc(lcd_t lcd, char c) {

	gpio_put(lcd.rs, true);

	for(uint8_t nibble = 0; nibble < 2; nibble++) {
	
		uint8_t n = (nibble)? c & 0x0f : c >> 4;
	
		lcd_put(lcd, n);
	}
}


void lcd_puts(lcd_t lcd, const char* str) {

	while(*str) {

		lcd_putc(lcd, *str);

		str++;
	}
}

void lcd_clear(lcd_t lcd) {

	lcd_put_command(lcd, 0x00);

	lcd_put_command(lcd, 0x01);

	sleep_ms(4);
}

void lcd_go_to_xy(lcd_t lcd, uint8_t x, uint8_t y) {

	uint8_t aux;

	if(y == 0) {

		aux = 0x40 + x;

		lcd_put_command(lcd, aux >> 4);

		lcd_put_command(lcd, aux & 0x0F);
	}

	else if (y == 1) {

		aux = 0xC0 + x;;

		lcd_put_command(lcd, aux >> 4);

		lcd_put_command(lcd, aux & 0x0F);
	}
}
