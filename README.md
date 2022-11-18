# Shopware 6 Tweaks
  
### Repositories
```php
$criteria = new Criteria();
$criteria->addFilter(new EqualsAnyFilter('[DATA]', [<DATA_ARRAY>]));
$criteria->addAssociation('field');
$criteria->addAssociation('field.attribute');

/** @var Entity $entity */
$entity = EntityRepositoryInterface $repository->search($criteria, $this->context);
```

### Setup Custom Fields
Initialize Custom Field on Install

```php
public function install(InstallContext $installContext): void
{
	/** @var EntityRepository $customFieldSetRepository */
	$customFieldSetRepository = $this->container->get('custom_field_set.repository');

	$customFieldSetRepository->upsert([
		[
			'name' => '[prefix_custom_field_set]',
			'config' => [
				'label' => [
					'de-DE' => '[LABEL]',
					'en-GB' => '[LABEL]'
				]
			],
			'customFields' => [
				[
					'name' => '[prefix_customfield_name]',
					'type' => CustomFieldTypes::[TYPE],
					'config' => [
						'label' => [
							'de-DE' => '[LABEL]',
							'en-GB' => '[LABEL]'
						],
						'type' => '[TYPE]',
						'customFieldType' => '[TYPE]',
						'customFieldPosition' => 1
					]
				]
			],
			'relations' => [
				[
					'id' => Uuid::randomHex(),
					'entityName' => '[ENTITY_NAME]'
				]
			]
		]
	], $installContext->getContext());
}
```

Delete Custom Field on Uninstall

```php
public function uninstall(UninstallContext $uninstallContext): void
{
	$connection = $this->container->get(Connection::class);

	$connection->executeUpdate('
		DELETE FROM `custom_field_set`
		WHERE name LIKE \'[prefix_custom_field_set]\'
	');

	$connection->executeUpdate('
		DELETE FROM `custom_field`
		WHERE name LIKE \'[prefix_customfield_name]\'
	');
}
```

Fill Custom Field
```php
public function setCustomField(string $param): void  
{  
	$criteria = new Criteria();  
	$criteria->addFilter(new EqualsAnyFilter('[IDENTIFIER]', [$param]));  
  
	/** @var Entity $entity */  
	$entity = $this->repository->search($criteria, $this->context)->first();  
  
	$this->repository->update([  
		[  
			'id' => $entity->getId(),  
			'customFields' => ['[prefix_customfield_name]' => '[data]']  
		]
	], $this->context);  
}
```

### Setup Page Subscription

```php
public static function getSubscribedEvents(): array  
{  
	return [  
		[...Page]LoadedEvent::class => 'on[...]Page'  
	];  
}
```

### vueJS Compiling StoreLocator
```bash
cd	|| ~/gitrepos/projectsSW6/buefa_shop/shop/plugins/NetiNextStoreLocator/src/Resources/app/storefront/src/plugins/store-locator/src/shared/pages/
cp to	|| ~/gitrepos/projectsSW6/buefa/src/plugins/NetiNextStoreLocator/src/Resources/app/storefront/src/plugins/store-locator/src/shared/pages/
// Edit JS file in IDE to modify filteredStoresByCountry (Project => Deploy on Local Machine & Server)
file: index.js

compile || /var/www/html/custom/plugins/NetiNextStoreLocator/src/Resources/app/storefront/src/plugins/store-locator
cd	|| ~/gitrepos/projectsSW6/buefa/src/plugins/NetiNextStoreLocator/src/Resources/app/storefront/src/plugins/store-locator/dist
cp to	|| ~/gitrepos/projectsSW6/buefa_shop/shop/plugins/NetiNextStoreLocator/src/Resources/app/storefront/src/plugins/store-locator/dist
// VUE Component Minified JS => Compile Files with command (Compile on Local Machine => Deploy file on Server & Project)
file: store-locator.js
command: npm run build

cd	|| ~/gitrepos/projectsSW6/buefa/src/plugins/NetiNextStoreLocator/src/Resources/app/storefront/dist/storefront/js
cp to	|| ~/gitrepos/projectsSW6/buefa_shop/shop/plugins/NetiNextStoreLocator/src/Resources/app/storefront/dist/storefront/js
// StoreLocator Plugin Minified JS => Compile Second (Local Machine => Deploy on Server & Project)
file: neti-next-store-locator.js
command: ./bin/build-storefront.sh

// Compile Theme with new minified JS files from local machine and clear cache (Server)
command: /opt/plesk/php/7.4/bin/php -d memory_limit=-1 bin/console theme:compile
command: /opt/plesk/php/7.4/bin/php -d memory_limit=-1 bin/console cache:clear
```
