<Picker
  label={'Account Type'}
  marginTop={16}
  value={accountType}
  items={stripeBankAccountType}
  placeholder={'Select account type'}
  onValueChange={this.onAccountTypeChange}
/>

<InputWithLabel
  label={'Account Number'}
  value={accountNumber}
  onValueChange={this.onValueC('accountNumber')}
  inputType={'numeric'}
/>

<Button
  label={'Add bank account'}
    containerMarginHorizontal={15}
    containerMarginTop={32}
    onPress={this.addBankAccount}
/>

<HeaderComponent
  navigation={navigation}
  text={
    orderId ? strings.transectionHistorystrings.transections
  }
  textStyle={{ left: 10 }}
  thirdIcon={null}
/>