# Работа #2. Нужно больше золота
Отчет по лабораторной работе #2 выполнил:
- Афонасьев Артём Ильич
- РИ-230948

Отметка о выполнении заданий:

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 0 |
| Задание 2 | * | 0 |
| Задание 3 | * | 0 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения.
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
Предложите вариант изменения найденных переменных для 10 уровней в игре. Визуализируйте изменение уровня сложности в таблице. 
  ![image](https://github.com/user-attachments/assets/dbf3d53e-5950-4861-9a35-e0970067d354)

https://docs.google.com/spreadsheets/d/14RSOAxw1mm4a4j-Xl_9-fDjyjH-trf0hZd1LNrxqT2c/edit?gid=2018496781#gid=2018496781
  
  

## Задание 2
Задание 2. Создайте 10 сцен на Unity с изменяющимся уровнем сложности
```c#
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.Networking;
using SimpleJSON;
using UnityEditor.SearchService;

public class DifficultyManager : MonoBehaviour
{
    // Ссылка на таблицу и ключ API
    private string sheetUrl = "https://sheets.googleapis.com/v4/spreadsheets/14RSOAxw1mm4a4j-Xl_9-fDjyjH-trf0hZd1LNrxqT2c/values/Лист3?key=AIzaSyDz74ovY2L7TNbHbQEuAUmn9vNWFIjg4XA";

    public float speed = 1f;
    public float timeBetweenEggDrops = 1f;
    public float chanceDirection = 0.1f;
    public int numEnergyShield = 3;
    public float EggAcceleration = 1;

    void Awake()
    {
        var currentSceneName = SceneManager.GetActiveScene().name;
        int level = ParseLevelFromSceneName(currentSceneName);
        StartCoroutine(GoogleSheets(level));
    }

    private int ParseLevelFromSceneName(string sceneName)
    {
        string[] parts = sceneName.Split('_');
        if (parts.Length > 1 && int.TryParse(parts[1], out int level))
        {
            return level;
        }
        else
        {
            Debug.LogError("Invalid scene name format.");
            return 1;  // Если имя сцены невалидное, по умолчанию используем уровень 1
        }
    }

    // Считывание данных из Google Sheets и применение их
    IEnumerator GoogleSheets(int level)
    {
        UnityWebRequest curentResp = UnityWebRequest.Get(sheetUrl);
        yield return curentResp.SendWebRequest();

        if (curentResp.result != UnityWebRequest.Result.Success)
        {
            Debug.LogError($"Error fetching data: {curentResp.error}");
            yield break;
        }

        string rawResp = curentResp.downloadHandler.text;

        if (string.IsNullOrEmpty(rawResp))
        {
            Debug.LogError("Response is empty.");
            yield break;
        }

        var rawJson = JSON.Parse(rawResp);

        if (rawJson == null || !rawJson.HasKey("values"))
        {
            Debug.LogError("Invalid JSON structure or missing 'values' key.");
            yield break;
        }

        var values = rawJson["values"];

        for (int i = 1; i < values.Count; i++) // Пропуск заголовков
        {
            try
            {
                var selectRow = values[i].AsArray;
                if (selectRow.Count >= 5)
                {
                    int key = int.Parse(selectRow[0]);
                    if (key == level)
                    {
                        float speedValue = float.Parse(selectRow[1]);
                        float timeBetweenEggDropsValue = float.Parse(selectRow[2]);
                        float chanceDirectionValue = float.Parse(selectRow[3]);
                        int numEnergyShieldValue = int.Parse(selectRow[4]);
                        float eggAccValue = float.Parse(selectRow[5]);

                        SetDifficulty(speedValue, timeBetweenEggDropsValue, chanceDirectionValue, numEnergyShieldValue, eggAccValue);
                    }
                }
                else
                {
                    Debug.LogWarning($"Skipping row {i}, insufficient data.");
                }
            }
            catch (System.Exception ex)
            {
                Debug.LogError($"Error processing row {i}: {ex.Message}");
            }
        }
    }
    
    private void SetDifficulty(float speedValue, float timeBetweenEggDropsValue, float chanceDirectionValue, int numEnergyShieldValue, float eggAccValue)
    {
        speed = speedValue;
        timeBetweenEggDrops = timeBetweenEggDropsValue;
        chanceDirection = chanceDirectionValue;
        numEnergyShield = numEnergyShieldValue;
        EggAcceleration = eggAccValue;


        UpdateGameParameters();
    }

    private void UpdateGameParameters()
    {
        GameObject enemy = GameObject.Find("Enemy");
        if (enemy != null)
        {
            EnemyDragon enemyDragon = enemy.GetComponent<EnemyDragon>();
            enemyDragon.speed = speed;
            enemyDragon.timeBetweenEggDrops = timeBetweenEggDrops;
            enemyDragon.chanceDirection = chanceDirection;
        }

        GameObject dragonPicker = GameObject.Find("dragonPicker");
        if (dragonPicker != null)
        {
            DragonPicker picker = dragonPicker.GetComponent<DragonPicker>();
            picker.numEnergyShield = numEnergyShield;
            picker.Starting();
        }
    }
}

```
![image](https://github.com/user-attachments/assets/5158a68f-4fd9-4526-93f4-8792fd9e81f1)


## Задание 3
Решение в 80+ баллов должно визуализировать данные из google-таблицы, и с помощью Python передавать в проект Unity. В Python данные также должны быть визуализированы.
![image](https://github.com/user-attachments/assets/af7312a4-fea4-4256-ad40-1039983b213a)

```pyton
import gspread
import numpy as np
import random
import matplotlib.pyplot as plt

# Подключение к Google Sheets
gc = gspread.service_account(filename='unitydatasience-445013-5a4a164d9ecf.json')
sh = gc.open("Workshop2")  # Название таблицы
worksheet = sh.worksheet('Лист3')

worksheet.clear()

# Формируем данные для записи в таблицу
data = [
    ["Level", "Speed", "TimeBetweenEggDrops", "ChanceDirection", "NumEnergyShield", "EggAcceleration"],
    [1, 1.0, 3.00, 0.0100, 5, 1.00],
    [2, 2.0, 2.72, 0.0144, 4, 1.22],
    [3, 3.0, 2.44, 0.0189, 4, 1.44],
    [4, 4.0, 2.17, 0.0233, 3, 1.67],
    [5, 5.0, 1.89, 0.0278, 3, 1.89],
    [6, 6.0, 1.61, 0.0322, 2, 2.11],
    [7, 7.0, 1.33, 0.0367, 2, 2.33],
    [8, 8.0, 1.06, 0.0411, 2, 2.56],
    [9, 9.0, 0.78, 0.0456, 2, 2.78],
    [10, 10.0, 0.50, 0.0500, 1, 3.00]
]

levels = [row[0] for row in data[1:]]
speed = [row[1] for row in data[1:]]
time_between_egg_drops = [row[2] for row in data[1:]]
chance_direction = [row[3] for row in data[1:]]
num_energy_shield = [row[4] for row in data[1:]]
egg_acceleration = [row[5] for row in data[1:]]

# Запись данных в таблицу
worksheet.update('A1', data)

# Визуализация изменения параметров
plt.figure(figsize=(10, 6))

# Графики для всех параметров
plt.subplot(2, 3, 1)
plt.plot(levels, speed, marker='o', label='Speed')
plt.title('Speed by Level')
plt.xlabel('Level')
plt.ylabel('Speed')
plt.grid(True)

plt.subplot(2, 3, 2)
plt.plot(levels, time_between_egg_drops, marker='o', label='Time Between Egg Drops', color='orange')
plt.title('Time Between Egg Drops by Level')
plt.xlabel('Level')
plt.ylabel('Time (s)')
plt.grid(True)

plt.subplot(2, 3, 3)
plt.plot(levels, chance_direction, marker='o', label='Chance Direction', color='green')
plt.title('Chance Direction by Level')
plt.xlabel('Level')
plt.ylabel('Chance')
plt.grid(True)

plt.subplot(2, 3, 4)
plt.plot(levels, num_energy_shield, marker='o', label='Num Energy Shield', color='red')
plt.title('Number of Energy Shields by Level')
plt.xlabel('Level')
plt.ylabel('Shields')
plt.grid(True)

# Визуализация Egg Acceleration
plt.subplot(2, 3, 5)
plt.plot(levels, egg_acceleration, marker='o', label='Egg Acceleration', color='purple')
plt.title('Egg Acceleration by Level')
plt.xlabel('Level')
plt.ylabel('Acceleration')
plt.grid(True)

plt.tight_layout()
plt.savefig('level_visualization.png')
plt.show()

print(f"Данные успешно записаны в таблицу: {sh.url}")

```
![image](https://github.com/user-attachments/assets/0cb558e3-71ad-48a2-9fbf-8400531bcdb1)



## Выводы
В ходе работы был разработан баланс изменения сложности для игры Dragon Picker, обеспечивающий плавное и логичное увеличение трудности на всех уровнях, учитывая ключевые параметры игры. Это позволяет поддерживать интерес игрока на протяжении всех 10 уровней



| Plugin | README |
| ------ | ------ |
| GitHub | [plugins/github/README.md][PlGh] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
