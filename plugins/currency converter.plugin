import requests
from base_plugin import BasePlugin, HookResult, HookStrategy


__id__ = "extera-converter"
__name__ = "⚖️ exteraConverter"
__description__ = "Довольно точный конвертер, работающий со множеством валют.\nКоманда по умолчанию [//cc].\n\nBy exteraBin Team"
__author__ = "@MrRighter"
__version__ = "1.0.2"
__icon__ = "exteraBin_Team/0"
__min_version__ = "11.12.0"



class ExteraConverter(BasePlugin):
    def on_plugin_load(self):
        self.add_on_send_message_hook()

    def on_send_message_hook(self, account: int, params) -> HookResult:
        if not isinstance(params.message, str) or not params.message.startswith("//cc"):
            return HookResult()

        try:
            amount = float(params.message.strip().split()[1].replace(",", "."))
            from_currency = params.message.strip().split()[2].upper()
            to_currency = params.message.strip().split()[3].upper()
        except Exception:
            error_msg = (
                f"{__name__}\n\n"
                "❌ Ошибка: недостаточно данных или они введены неверно\n\n"
                "✏️ Пример правильного использования:\n//cc 100 USD RUB"
            )
            params.message = error_msg
            return HookResult(strategy=HookStrategy.MODIFY, params=params)

        if from_currency == to_currency:
            error_msg = (
                f"{__name__}\n\n"
                "❌ Ошибка: Введены одинаковые валюты"
            )
            params.message = error_msg
            return HookResult(strategy=HookStrategy.MODIFY, params=params)

        result = f"{__name__}\n\n"
        try:
            # Прямая конвертация фиатных валют
            data = requests.get("https://www.cbr-xml-daily.ru/daily_json.js").json()

            # Конвертация из валюты в валюту относительно рубля
            if len({from_currency, to_currency, "RUB"}) == 3:
                value_from_currency = data["Valute"][from_currency]["Value"]
                nominal_from_currency = data["Valute"][from_currency]["Nominal"]
                value_to_currency = data["Valute"][to_currency]["Value"]
                nominal_to_currency = data["Valute"][to_currency]["Nominal"]

                currency_to_currency = float(amount * value_from_currency / nominal_from_currency / value_to_currency / nominal_to_currency)
                result += f"{amount} {from_currency} = {currency_to_currency} {to_currency}"
            else:
                if from_currency == "RUB":  # Из рубля в валюту
                    value_to_currency = data["Valute"][to_currency]["Value"]
                    nominal_to_currency = data["Valute"][to_currency]["Nominal"]

                    rub_to_currency = float(1 / value_to_currency * nominal_to_currency * amount)
                    result += f"{amount} {from_currency} = {rub_to_currency} {to_currency}"
                elif to_currency == "RUB":  # Из валюты в рубль
                    value_from_currency = data["Valute"][from_currency]["Value"]
                    nominal_from_currency = data["Valute"][from_currency]["Nominal"]

                    currency_to_rub = float(value_from_currency / nominal_from_currency * amount)
                    result += f"{amount} {from_currency} = {currency_to_rub} {to_currency}"

            params.message = result
            return HookResult(strategy=HookStrategy.MODIFY, params=params)
        except KeyError:
            result += (
                "❌ Ошибка: Некорректный и/или неподдерживаемый ввод валют(-ы).\n\n"
                "✏️ Пример правильного использования:\n//cc 100 USD RUB"
            )
            params.message = result
            return HookResult(strategy=HookStrategy.MODIFY, params=params)
