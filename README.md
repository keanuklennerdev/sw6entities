### How to use EntityRepositories
#### In Service
```php
use Shopware\Core\Content\Product\ProductEntity;
use Shopware\Core\Framework\DataAbstractionLayer\EntityRepositoryInterface;

$criteria = new Criteria();
$criteria->addFilter(new EqualsAnyFilter('id', $id));
$criteria->addAssociation('field');
$criteria->addAssociation('field.attribute');

/** @var ProductEntity $product */
$product = $this->productRepository->search($criteria, $this->context);

$this->productRepository->update([
	[
		'id' => $productId,
		'modifiedField' => $modifiedField
	]
], $this->context);
```
#### services.xml
```xml
<!--Subscriber-->
<service id="Prefix\PluginName\Subscriber\SubscriberName">
    <tag name="kernel.event_subscriber"/>
    <argument type="service" id="Prefix\PluginName\Service\ServiceName"/>
</service>

<!--Services-->
<service id="Prefix\PluginName\Service\ServiceName">
    <argument type="service" id="product.repository"/>
    <argument type="service" id="Shopware\Core\System\SystemConfig\SystemConfigService"/>
</service>
```
### Custom Field in Shopware Plugin
#### Init custom field on plugin install:
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
#### Custom field particularities
##### Text editor as custom field [config values are important, HTML type is not enough]
```php
'name' => '[prefix_customfield_name]',
'type' => CustomFieldTypes::HTML,
'config' => [
	'label' => [
		'de-DE' => '[LABEL]',
		'en-GB' => '[LABEL]'
	],
	'type' => 'HTML',
	'componentName' => 'sw-text-editor',
	'customFieldType' => 'textEditor',
	'customFieldPosition' => 1
]
```
#### Delete custom field on plugin uninstall:
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
#### Fill custom field in plugin:
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
