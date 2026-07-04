### Нет защиты от повторной отправки формы (race condition)

**Симптом.** При медленной сети (или если `setTimeout` в моке увеличить до 3–5 секунд) пользователь может нажать кнопку «Записаться» несколько раз подряд или быстро нажать Enter в поле ввода. В результате отправляется **несколько параллельных запросов** к API, и на экране появляется несколько сообщений об успехе/ошибке, а кнопка «Отправка…» моргает. В реальном приложении это приведёт к дублированию броней.

**Причина.** Флаг `isSubmitting` отсутствует. Кнопка блокируется через `disabled`, но между первым кликом и применением `disabled` может сработать второй клик (особенно на медленных устройствах). Также Enter в поле ввода вызывает `submit`, который не проверяет состояние отправки.

**Исправление.** Добавить флаг `isSubmitting` и проверять его в начале `handleFormSubmit`:

// State

let currentSlots = [];

let isModalOpen = false;

let selectedSlot = null;

let autoCloseTimer = null;

let isSubmitting = false; // ← добавить

async function handleFormSubmit(formData) {

  if (isSubmitting) return; // ← добавить защиту

  

  const validation = validateForm(formData);

  if (!validation.isValid) {

    showValidationErrors(validation.errors);

    return;

  }

  

  isSubmitting = true; // ← добавить

  showValidationErrors({});

  clearMessages();

  setSubmitButtonLoading(true);

  

  try {

    const result = await createBooking(formData, false);

    

    showMessage(

      `Вы успешно записаны на мастер-класс! Ваш номер брони: ${result.data.bookingId}`,

      'success'

    );

    

    autoCloseTimer = setTimeout(() => {

      closeModal();

    }, 2000);

    

  } catch (error) {

    if (error.status === 409) {

      showMessage(error.message || 'Вы уже записаны на это занятие', 'error');

    } else if (error.status === 410) {

      showMessage(error.message || 'Место на мастер-классе закончилось', 'error');

    } else {

      showMessage(error.message || 'Произошла ошибка. Попробуйте позже.', 'error');

    }

    

    setSubmitButtonLoading(false);

  } finally {

    isSubmitting = false; // ← добавить

  }

}