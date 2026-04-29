using UnityEngine;
using TMPro;
using UnityEngine.UI;

/// Основной игровой интерфейс (HUD).
/// Отображает информацию о растении, погоде и управлении.
public class GameUI : MonoBehaviour
{
    [Header("Main HUD")]
    [SerializeField] private GameObject mainHUD;            // Основной HUD
    [SerializeField] private TextMeshProUGUI plantNameText; // Название растения
    [SerializeField] private TextMeshProUGUI healthText;    // Здоровье растения
    [SerializeField] private Image healthBar;               // Полоска здоровья

    [Header("Environment Info")]
    [SerializeField] private TextMeshProUGUI lightText;     // Освещение
    [SerializeField] private TextMeshProUGUI windText;      // Ветер
    [SerializeField] private TextMeshProUGUI tempText;      // Температура

    [Header("Status Indicators")]
    [SerializeField] private GameObject lightIndicator;     // Индикатор освещения
    [SerializeField] private GameObject windIndicator;      // Индикатор ветра
    [SerializeField] private GameObject warningPanel;       // Панель предупреждений

    [Header("Controls")]
    [SerializeField] private GameObject controlsPanel;      // Панель управления
    [SerializeField] private Button diagnosticButton;       // Кнопка диагностики
    [SerializeField] private Button settingsButton;         // Кнопка настроек
    [SerializeField] private Button menuButton;             // Кнопка меню

    private PlantData currentPlant;  // Текущее растение

    // ИНИЦИАЛИЗАЦИЯ
    private void Start()
    {
        // Настраиваем кнопки
        SetupButtons();

        // Показываем подсказки управления при старте
        ShowControlsHint();
    }
    private void Update()
    {
        // Обновляем информацию о среде
        UpdateEnvironmentInfo();

        // Обработка горячих клавиш
        HandleHotkeys();
    }
    // НАСТРОЙКА
    private void SetupButtons()
    {
        if (diagnosticButton != null)
            diagnosticButton.onClick.AddListener(OpenDiagnostic);

        if (settingsButton != null)
            settingsButton.onClick.AddListener(OpenSettings);

        if (menuButton != null)
            menuButton.onClick.AddListener(OpenMainMenu);
    }
    // ОБНОВЛЕНИЕ ИНФОРМАЦИИ
    /// Обновляет информацию об окружающей среде каждый кадр.
    private void UpdateEnvironmentInfo()
    {
        // Освещение
        if (LightSystem.Instance != null && currentPlant != null)
        {
            float light = LightSystem.Instance.GetInsolationForPlant(transform);
            lightText.text = $"Свет: {light:P0}";

            // Обновляем индикатор (цвет)
            UpdateIndicator(lightIndicator, light, currentPlant.MinLight, currentPlant.MaxLight);
        }
        // Ветер
        if (WindManager.Instance != null)
        {
            windText.text = $"Ветер: {WindManager.Instance.CurrentWindSpeed:F1} м/с";
        }

        // Температура (заглушка - замените на реальную)
        float currentTemp = 20f;
        tempText.text = $"Температура: {currentTemp}°C";
    }
    /// Обновляет цвет индикатора на основе значений.
    private void UpdateIndicator(GameObject indicator, float value, float min, float max)
    {
        if (indicator == null) return;

        Image indicatorImage = indicator.GetComponent<Image>();
        if (indicatorImage == null) return;

        if (value < min)
            indicatorImage.color = Color.blue;      // Слишком мало
        else if (value > max)
            indicatorImage.color = Color.red;       // Слишком много
        else
            indicatorImage.color = Color.green;     // Норма
    }

    // УПРАВЛЕНИЕ
    /// Обработка горячих клавиш.
    private void HandleHotkeys()
    {
        // D - диагностика
        if (Input.GetKeyDown(KeyCode.D))
        {
            OpenDiagnostic();
        }

        // Escape - пауза/меню
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            TogglePause();
        }

        // H - показать подсказки
        if (Input.GetKeyDown(KeyCode.H))
        {
            ShowControlsHint();
        }
    }
    /// Открывает панель диагностики.
    public void OpenDiagnostic()
    {
        DiagnosticPanel diagnosticPanel = FindObjectOfType<DiagnosticPanel>();
        if (diagnosticPanel != null && currentPlant != null)
        {
            diagnosticPanel.ShowPlantInfo(currentPlant);
        }
    }
    /// Открывает настройки.
    public void OpenSettings()
    {
        Debug.Log("Открыть настройки");
    }
    /// Возвращается в главное меню.

    public void OpenMainMenu()
    {
        UnityEngine.SceneManagement.SceneManager.LoadScene("MainMenu");
    }
    /// Переключает паузу.

    private void TogglePause()
    {
        Time.timeScale = Time.timeScale == 0 ? 1 : 0;
    }

    // УВЕДОМЛЕНИЯ
 
    /// Показывает уведомление.
    public void ShowNotification(string message, NotificationType type = NotificationType.Info)
    {
        // Используем NotificationSystem
        if (NotificationSystem.Instance != null)
        {
            // Вызываем правильный метод в зависимости от типа
            switch (type)
            {
                case NotificationType.Info:
                    NotificationSystem.Instance.ShowInfo(message);
                    break;
                case NotificationType.Success:
                    NotificationSystem.Instance.ShowSuccess(message);
                    break;
                case NotificationType.Warning:
                    NotificationSystem.Instance.ShowWarning(message);
                    break;
                case NotificationType.Error:
                    NotificationSystem.Instance.ShowError(message);
                    break;
            }
        }
        else
        {
            // Если NotificationSystem нет - выводим в консоль
            Debug.Log($"[{type}] {message}");
        }
    }

    /// Показывает подсказки по управлению.
    private void ShowControlsHint()
    {
        string hint = "Управление:\n" +
                     "D - Диагностика\n" +
                     "Esc - Пауза\n" +
                     "H - Помощь\n" +
                     "Колёсико мыши - Зум";

        ShowNotification(hint, NotificationType.Info);
    }

    /// Устанавливает текущее растение.
    public void SetCurrentPlant(PlantData plant)
    {
        currentPlant = plant;
        plantNameText.text = plant.PlantName;
    }
}
