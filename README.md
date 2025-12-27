/*
* Автор: Солдатов В.И.
*/

/*
#* Основной модуль программы анализа функции.
* Содержит функции для вычисления значения в точке, табулирования,
* поиска экстремумов, поиска аргумента по значению функции и вычисления производной.
*/

#define _CRT_SECURE_NO_DEPRECATE

#include <stdio.h>

#include <math.h>

#include <locale.h>

#define PI 3.14159265358979323846 

#define MAX_ITERATIONS 16

double factorial(int n);

double f(double x);

void tabulate(double start, double end, double step);

double findX(double Y, double start, double end, double step);

double derivative(double x, double h);

double find_minimum(double start, double end, double step);

double find_maximum(double start, double end, double step);

void log_error(const char* message);

int main() {

	setlocale(LC_CTYPE, "RUS");

	printf("                                                                     \n");
	printf("                   **************************************************\n");
	printf("                   *                                                *\n");
	printf("                   *                  Курсовой проект               *\n");
	printf("                   *     Конструирование программы анализа функции  *\n");
	printf("                   *              Выполнил: Солдатов В.И.           *\n");
	printf("                   *          Руководитель: Минакова О. В.          *\n");
	printf("                   *                Группа: бИЦ-251                 *\n");
	printf("                   *                                                *\n");
	printf("                   **************************************************\n");
	printf("                                                                     \n");

	int choice;
	double x;
	double start;
	double end;
	double step;
	double Y;
	double h;

	do {

		printf("\n------------Меню------------\n");
		printf("1. Значение функции в точке\n");
		printf("2. Таблица значений\n");
		printf("3. Поиск минимума/максимума \n");
		printf("4. Поиск X по Y \n");
		printf("5. Производная в точке\n");
		printf("0. Выход\n");
		printf("Выберите пункт: ");

		scanf("%d", &choice);
		switch (choice) {
		case 1:
			printf("\nВведите x: ");
			scanf("%lf", &x);
			double result = f(x);
			printf("f(%.10f) = %.10f\n", x, result);
			break;
		case 2:
			printf("\nВведите начало интервала: ");
			scanf("%lf", &start);
			printf("Введите конец интервала: ");
			scanf("%lf", &end);
			printf("Введите шаг: ");
			scanf("%lf", &step);
			if (start > end) {
				printf("Ошибка: начало интервала должно быть меньше конца!\n");
				break;
			}
			tabulate(start, end, step);
			break;
		case 3:
			printf("\nВведите начало отрезка: ");
			scanf("%lf", &start);
			printf("Введите конец отрезка: ");
			scanf("%lf", &end);
			printf("Введите шаг поиска: ");
			scanf("%lf", &step);
			double min_val = find_minimum(start, end, step);
			double max_val = find_maximum(start, end, step);
			printf("\n===Экстремумы на отрезке [%.2f,%.2f]===\n", start, end);
			printf("Минимум:%.10f\n", min_val);
			printf("Максимум:%.10f\n", max_val);
			break;
		case 4:
			printf("\nВведите значение Y: ");
			scanf("%lf", &Y);
			printf("Введите начало интервала поиска: ");
			scanf("%lf", &start);
			printf("Введите конец интервала поиска: ");
			scanf("%lf", &end);
			printf("Введите шаг поиска: ");
			scanf("%lf", &step);
			if (start > end) {
				printf("Ошибка: начало интервала должно быть меньше конца!\n");

				break;
			}
			double result_x = findX(Y, start, end, step);
			if (result_x != -1.0) {
				printf("Найден x = %.10f\n", result_x);

			}

			break;
		case 5:
			printf("\nВведите x для вычисления производной: ");
			scanf("%lf", &x);

			printf("Введите шаг h: ");
			scanf("%lf", &h);

			double deriv = derivative(x, h);

			printf("f'(%.10f) = %.10f\n", x, deriv);

			break;

		case 0:
			printf("Завершение работы программы.\n");
			break;
		default:
			printf("Неверный выбор. Попробуйте снова.\n");
		}
	} while (choice != 0);

	return 0;
}

/*
* записывает сообщение об ошибке в файл
* message - сообщение об ошибке
* открывает файл errors.log в режиме добавления и записывает сообщение
*/

void log_error(const char* message) {

	FILE* log_file = fopen("errors.log", "a");

	if (log_file) {
		fprintf(log_file, "ERROR: %s\n", message);
		fclose(log_file);
	}
}

/*
* вычисляет факторил числа n
* n - целое неотрицательное число 
* возвращает вещественное число
* в случае отрицательного n записывает ошибку в log и возвращает -1.0
*/

double factorial(int n) {
	if (n < 0) {
		log_error("Отрицательный факториал");
		return -1.0;
	}

	
	double result = 1.0;
	for (int i = 2; i <= n; i++) {
		result *= i;
	}
	return result;
}

/*
* вычисляет значение функции f(x) в точке x 
* x - аргумент функции
* возвращает значение функции согласно кусочному определению
*/

double f(double x) {

	if (x < 1) {

		return (x * x - 4) / (x - 2);
	}

	else if (x < PI / 2) {

		return (1.0 / cos(x)) - tan(x);
	}
	else {
		double sum = 0;
		for (int n = 0; n <= 15; n++) {
			double slagaemoe = pow(-1, n) * pow(x, 3 * n) / factorial(3 * n + 1);
			sum += slagaemoe;
		}
		return sum;
	}
	return 0;
}
/*
* табулирует функцию на заданном интервале с заданным шагом
* start - начало интервала
* end - конец интервала
* step - шаг табуляции
* выводит таблицу значений в консоль 
* в случае некорректных параметров записывает ошибку в log
*/

void tabulate(double start, double end, double step) {
	if (start > end) {
		log_error("Начало интервала больше конца:");
		printf("Ошибка: начало интервала не может быть больше конца\n");
		return;
	}

	printf("\n===Таблица значений===\n");
	printf("\n");
	printf("| x | f(x) |\n");
	printf("|--------|--------| \n");

	for (double x = start; x <= end; x += step) {
		double y = f(x);
		printf("|%-8.2f|%-8.2f|\n", x, y);
	}
}
/*
* ищет значение x, при котором f(x) приближенно равно Y на заданном интервале
* Y - искомое значение функции
* start - начало интервала поиска
* end - конец интервала поиска
* step - шаг поиска
* возвращает найденное x или -1.0, если решение не найдено
*/
double findX(double Y, double start, double end, double step) {
	printf("\n=== Поиск x: f(x) = %.6f ===\n", Y);

	for (double x = start; x <= end; x += step) {
		double y = f(x);

		if (fabs(y - Y) < 0.1) {

			return x;
		}
	}

	printf("Решение не найдено на отрезке [%.2f, %.2f]\n", start, end);
	return -1.0;
}
/*
* вычисляет производную функции в точке x методом центральной разности
* x - точка, в которой вычисляется производная 
* h - шаг для численного дифференцирования
* возвращает приближенное значение производной
* в случае слишком маленького h выводит сообщение об ошибке и возвращает -1.0
*/
double derivative(double x, double h) {
	if (h < 0.1) {
		printf("Ошибка: шаг должен быть не менее 0.1\n");
		return -1.0;
	}

	return (f(x + h) - f(x - h)) / (2 * h);
}
/*
* находит минимальное значение функции на отрезке 
* start - начало отрезка
* end - конец отрезка
* step - шаг поиска
* возвращает минимальное значение функции на отрезке
*/ 
double find_minimum(double start, double end, double step) {
	double min_val = INFINITY;

	for (double x = start; x <= end; x += step) {
		double y = f(x);
		if (y < min_val) {
			min_val = y;

		}
	}
	return min_val;
}
/*
* находит максимальное значение функции на отрезке
* start - начало отрезка
* end - конец отрезка
* step - шаг поиска
* возвращает максимальное значение функции на отрезке
*/
double find_maximum(double start, double end, double step) {
	double max_val = -INFINITY;

	for (double x = start; x <= end; x += step) {
		double y = f(x);
		if (y > max_val) {
			max_val = y;

		}
	}
	return max_val;

}
