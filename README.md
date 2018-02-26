<?php

/**
 * @file
 * Contains \Drupal\accordion_blog\Plugin\Block\AccordionBlog.
 */

namespace Drupal\accordion_blog\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Datetime\DateFormatterInterface;

/**
 * Provides a 'blog' block.
 *
 * @Block(
 *   id = "accordion_blog_1",
 *   admin_label = @Translation("Accordion Blog"),
 *   category = @Translation("Custom block")
 * )
 */
class AccordionBlog extends BlockBase {

    /**
     * {@inheritdoc}
     */
    public function build() {
        $language = \Drupal::languageManager()->getCurrentLanguage()->getId();
        $query = \Drupal::entityQuery('node');
        $query->condition('status', 1);
        $query->condition('langcode', $language);
        $query->condition('type', 'blog');
        $query->sort('created', 'ASC');
        $entity_ids = $query->execute();
        $nodes = entity_load_multiple('node', $entity_ids);
        $data = array();
        foreach ($nodes as $node) {
            $node = $node->getTranslation($language);
            //$date = new \DateTime($node->get('created')->getValue()[0]['value']);
            $timestamp = $node->created->value;

            $url = "/staff-blog/" . format_date($timestamp, 'custom', 'Y') . "/" . format_date($timestamp, 'custom', 'm');
            if (array_key_exists(format_date($timestamp, 'custom', 'Y'), $data)) {
                $data[format_date($timestamp, 'custom', 'Y')][format_date($timestamp, 'custom', 'Ym')]['count'] ++;
                $data[format_date($timestamp, 'custom', 'Y')][format_date($timestamp, 'custom', 'Ym')]['url'] = "<a href=$url>" . format_date($timestamp, 'custom', 'M', 'Asia/Tokyo', $language) . "</a>";
                ;
                continue;
            } else {
                $data[format_date($timestamp, 'custom', 'Y')][format_date($timestamp, 'custom', 'Ym')]['url'] = "<a href=$url>" . format_date($timestamp, 'custom', 'M', 'Asia/Tokyo', $language) . "</a>";
                $data[format_date($timestamp, 'custom', 'Y')][format_date($timestamp, 'custom', 'Ym')]['count'] = 1;
            }
        }
        $markup = "<div id='accordion'>";
        foreach ($data as $year => $rs) {
            $count = 0;
            $month_list = "<ul>";
            foreach ($rs as $nodedata) {
                // $url = $nodedata['url'] . "(" . $nodedata['count'] . ")";
                $url = $nodedata['url'];
                $month_list .= "<li>$url</li>";
                $count = $count + $nodedata['count'];
            }
            $month_list.="</ul>";
            $yurl = "/staff-blog/" . $year;
            // $singal_year = "<a href=$yurl>" . $year . "(" . $count . ")</a>";
            $singal_year = "<a href=$yurl>" . $year .t('年'). "</a>";
            $markup .= "<h3>" . $singal_year . "</h3>";
            $markup .= "<div>" . $month_list . "</div>";
        }
        $markup .= "</div>";
        return array(
            '#type' => 'markup',
            '#markup' => $markup,
            '#attached' => array(
                'library' => array('accordion_blog/accordion_blog.accordion'),
            ),
        );
    }

    /**
     * {@inheritdoc}
     */
    public function getCacheMaxAge() {
        return 0;
    }

}
