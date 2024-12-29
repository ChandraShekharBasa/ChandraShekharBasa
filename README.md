const sendAuthData = async (email, password, captchaText, csrf) => {
  isLoading(true);
  try {
    const response = await fetch('http://reminder-app.com/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json;charset=utf-8'
      },
      body: JSON.stringify({ email, password, captchaText, _csrf: csrf })
    });

    const result = await response.json();

    if (result.error !== null) {
      isLoading(false);
      openNotification(result.error, 'error');
      return;
    }

    goMain();
  } catch (error) {
    isLoading(false);
    openNotification('Something went wrong', 'error');
  }
};
