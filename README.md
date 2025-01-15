# Lab_06
## Цели работы:
Научиться работать с LINQ средствами языка C#.

## Задание №1
Зарегистрируйтесь на сайте https://openweathermap.org/ для получения ключа (API key) к API от сервиса погоды.
Создайте структуру Weather, содержащую свойства Country(страна), Name(город или название местности), Temp(температура воздуха), Description(описание погоды).
Используя API, получите не менее 50 значений текущей погоды в разных точках мира.
Используйте запрос вида: 
https://api.openweathermap.org/data/2.5/weather?lat={Широта}&lon={Долгота}&appid={API key}, где:
Широта - дробная величина в диапазоне от -90 до 90. 
Долгота - дробная величина в диапазоне от -180 до 180.
API key - ключ, полученный при регистрации на сайте https://openweathermap.org/.
Значения Широты и Долготы изменяйте случайным образом в заданных диапазонах, если для полученной координаты нет значения Country или Name, следует сгенерировать новые координаты.
На основе полученных данных создайте и заполните коллекцию структур Weather.
С помощью LINQ запросов к созданной коллекции, получите и выведите на консоль следующие данные:
1. Страну с максимальной и минимальной температурой.
2. Среднюю температуру в мире.
3. Количество стран в коллекции.
4. Первую найденную страну и название местности, в которых Description принимает значение: "clear sky","rain","few clouds"

## Код задания №1

    using System.Net.Http.Headers;
    using System.Runtime.InteropServices.JavaScript;
    using System.Text.Json.Nodes;
    
    // Структура Weather описывает погоду для определенного города.
    struct Weather
    {
        public string Country { get; set; }      // Свойство для хранения названия страны
        public string Name { get; set; }         // Свойство для хранения названия города
        public double Temp { get; set; }         // Свойство для хранения температуры
        public string Description { get; set; }  // Свойство для хранения описания погоды
    
        // Конструктор для инициализации полей структуры Weather.
        public Weather(string country, string name, double temp, string description)
        {
            Country = country;
            Name = name;
            Temp = temp;
            Description = description;
        }
    
        // Переопределенный метод ToString() для красивого вывода информации о погоде.
        public override string ToString()
        {
            return $"Country: {Country}, Name: {Name}, Temp: {Temp}, Description: {Description}\n";
        }
    }
    
    class Program
    {
        static void Main(string[] args)
        {
            string apiKey = " "; // Ключ API для доступа к OpenWeatherMap API
            string URL = $"https://api.openweathermap.org/data/2.5/weather"; // URL для получения данных о погоде
    
            Weather[] weathers = new Weather[50]; // Массив для хранения данных о погоде для 50 случайных мест
            Random random = new Random(); // Объект для генерации случайных координат
            int i = 0; // Счетчик успешно полученных данных
    
            // Цикл для запроса данных о погоде для случайных координат
            while (i < weathers.Length)
            {
                // Генерация случайных координат (широта и долгота) в пределах земного шара
                double minLatitude = -90;
                double maxLatitude = 90;
                double minLongtitude = -180;
                double maxLongtitude = 180;
                double latitude = random.NextDouble() * (maxLatitude - minLatitude) + minLatitude;
                double longtitude = random.NextDouble() * (maxLongtitude - minLongtitude) + minLongtitude;
    
                // Создание клиента для HTTP-запросов
                HttpClient client = new HttpClient();
                client.BaseAddress = new Uri(URL); // Установка базового адреса для клиента
                client.DefaultRequestHeaders.Clear(); // Очистка всех заголовков
                client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json")); // Установка заголовка для принятия JSON-ответа
    
                // Формирование строки с параметрами запроса
                string urlParameters = $"?lat={latitude}&lon={longtitude}&appid={apiKey}";
                HttpResponseMessage response = client.GetAsync(urlParameters).Result; // Выполнение запроса
    
                // Если запрос успешен, обрабатываем данные
                if (response.IsSuccessStatusCode)
                {
                    var data = response.Content.ReadAsStringAsync().Result;
                    var json = JsonObject.Parse(data); // Разбор и извлечение данных из строки JSON-ответа
    
                    // Заполнение полей структуры Weather данными из JSON-ответа
                    Weather res = new Weather();
                    res.Country = (string)json["sys"]["country"];
                    res.Name = (string)json["name"];
                    res.Temp = (double)json["main"]["temp"];
                    res.Description = (string)json["weather"][0]["main"];
    
                    // Проверка, что страна и название города не пусты
                    if (res.Country == "" || res.Name == "") continue;
                    else
                    {
                        weathers[i] = res; // Сохранение данных в массив
                        i += 1; // Увеличение счетчика
                    }
    
                    Console.WriteLine(i); // Вывод текущего количества успешных запросов
                }
                else
                {
                    Console.WriteLine("Not successful"); // Если запрос неудачен, выводим сообщение
                }
                client.Dispose(); // Освобождение ресурсов клиента
            }
            Console.WriteLine("Successful"); // Уведомление об успешном завершении цикла запросов
    
            // Поиск и вывод данных о минимальной температуре
            Console.WriteLine("Minimum temperature: ");
            Weather minTemp = (from w in weathers
                               orderby w.Temp
                               select w).First();
            Console.WriteLine(minTemp);
    
            // Поиск и вывод данных о максимальной температуре с помощью метода расширения
            Console.WriteLine("Maximum temperature: ");
            Weather maxTemp = weathers.OrderByDescending(w => w.Temp).First();
            Console.WriteLine(maxTemp);
    
            // Вычисление и вывод средней температуры
            Console.WriteLine("Average:");
            double average = weathers.Average(w => w.Temp);
            Console.WriteLine(average);
    
            // Подсчет количества уникальных стран в массиве
            Console.WriteLine("Quantity of countries: ");
            int quantity = weathers.GroupBy(w => w.Country).Count();
            Console.WriteLine(quantity);
    
            // Поиск первого места с ясной, дождливой или слегка облачной погодой
            var last = weathers.Where(w => w.Description == "Clear sky" || w.Description == "Rain" || w.Description == "Few clouds");
            if (last.Any())
            {
                Weather weather = last.First();
                Console.WriteLine("Point with clear sky or rain or clouds ");
                Console.WriteLine(weather);
            }
            else
            {
                Console.WriteLine("No such place found");
            }
    
            // Вывод полного списка показателей погоды
            Console.WriteLine("List of indications: ");
            for (int j = 0; j < weathers.Length; j++)
            {
                Console.WriteLine($"{j}: {weathers[j]}");
                Console.WriteLine();
            }
        }
    }

C# lab06
