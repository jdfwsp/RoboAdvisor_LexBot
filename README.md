# 🐍🤖 Robo Advisor for Retirement Plans 💰🐍

## Purpose
Improve customer experience for a prominent retirement plan provider with a robo advisor that could be used by customers or potential customers to get investment portfolio recommendations for retirement.

## Technology
* AWS Lex bot
* AWS Lambda function (Python 3.7)

![Bot Demonstration](RoboAdvisor/bot.gif)

## Method
Validate input from user
```
def validate_data(age, investment_amount, intent_request):
    # Validate that the user is between 0 - 65 yrs old
    if age is not None:
        age = parse_int(age)
        if age < 0 or age >= 65:
            return build_validation_result(
                False,
                "age",
                "You must be younger than 65 years old to use this service.",
            )
    
    # Validate that the amount is greater than 5000
    if investment_amount is not None:
        investment_amount = parse_float(investment_amount)
        if investment_amount < 5000:
            return build_validation_result(
                False,
                "investmentAmount",
                "You must invest more than $5000 to use this service.",
            )

    # Return True if age and amount are valid
    return build_validation_result(True, None, None)
```
Declare dictionary with risk level-specific blurbs for final bot output
```
risk = {
    "None": "100% bonds (AGG), 0% equities (SPY)",

    "Very Low": "80% bonds (AGG), 20% equities (SPY)",

    "Low": "60% bonds (AGG), 40% equities (SPY)",

    "Medium": "40% bonds (AGG), 60% equities (SPY)",

    "High": "20% bonds (AGG), 80% equities (SPY)",

    "Very High": "0% bonds (AGG), 100% equities (SPY)"
}
```
Use input to recommend portfolio
```
def recommend_portfolio(intent_request):
    """
    Performs dialog management and fulfillment for recommending a portfolio.
    """

    first_name = get_slots(intent_request)["firstName"]
    age = get_slots(intent_request)["age"]
    investment_amount = get_slots(intent_request)["investmentAmount"]
    risk_level = get_slots(intent_request)["riskLevel"]
    source = intent_request["invocationSource"]

    if source == "DialogCodeHook":
        # Perform basic validation on the supplied input slots.
        # Use the elicitSlot dialog action to re-prompt
        # for the first violation detected.

        # Gets all the slots
        slots = get_slots(intent_request)

        ### DATA VALIDATION CODE STARTS HERE ###
        validation_result = validate_data(age, investment_amount, intent_request)

        if not validation_result["isValid"]:
            slots[validation_result["violatedSlot"]] = None  # Cleans invalid slot

            # Returns an elicitSlot dialog to request new data for the invalid slot
            return elicit_slot(
                intent_request["sessionAttributes"],
                intent_request["currentIntent"]["name"],
                slots,
                validation_result["violatedSlot"],
                validation_result["message"],
            )

        ### DATA VALIDATION CODE ENDS HERE ###

        # Fetch current session attibutes
        output_session_attributes = intent_request["sessionAttributes"]

        return delegate(output_session_attributes, get_slots(intent_request))

    # Get the initial investment recommendation
    
    initial_recommendation = risk[risk_level]
    
    ### FINAL INVESTMENT RECOMMENDATION CODE STARTS HERE ###
    # Return a message with the initial recommendation based on the risk level.
    return close(
        intent_request["sessionAttributes"],
        "Fulfilled",
        {
            "contentType": "PlainText",
            "content": """{} thank you for your information;
            based on the risk level you defined, my recommendation is to choose an investment portfolio with {}
            """.format(
                first_name, initial_recommendation
            ),
        },
    )
    ```