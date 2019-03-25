# Crypt_PHP


I'm not aware of a PHP library with straightforward API for this.

I've implemented several libraries however that could help with the task. asn1, crypto-util and x509 are available via composer.

Here's a barebones proof of concept that extracts all certificates from a PKCS7 PEM file:

<?php

use ASN1\Element;
use ASN1\Type\Constructed\Sequence;
use CryptoUtil\PEM\PEM;
use X509\Certificate\Certificate;

require __DIR__ . "/vendor/autoload.php";

$pem = PEM::fromFile("path-to-your.p7b");
// ContentInfo: https://tools.ietf.org/html/rfc2315#section-7
$content_info = Sequence::fromDER($pem->data());
// SignedData: https://tools.ietf.org/html/rfc2315#section-9.1
$signed_data = $content_info->getTagged(0)->asExplicit()->asSequence();
// ExtendedCertificatesAndCertificates: https://tools.ietf.org/html/rfc2315#section-6.6
$ecac = $signed_data->getTagged(0)->asImplicit(Element::TYPE_SET)->asSet();
// ExtendedCertificateOrCertificate: https://tools.ietf.org/html/rfc2315#section-6.5
foreach ($ecac->elements() as $ecoc) {
    $cert = Certificate::fromASN1($ecoc->asSequence());
    echo $cert->toPEM() . "\n";
}
