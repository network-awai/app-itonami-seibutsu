(ns seibutsu.app-test
  (:require [cljs.test :refer [deftest is testing use-fixtures]]
            [re-frame.core :as rf]
            [re-frame.db :as rf-db]
            [seibutsu.app :as app]))

(use-fixtures :each
  {:before (fn [] (reset! rf-db/app-db {}))})

(deftest initial-db-matches-original-svelte-data-test
  (testing "default-db carries the exact values the Svelte +page.svelte <script> block held"
    (is (= "Seibutsu S3ibtsu1" (:title app/default-db)))
    (is (= "etzhayyim-project-seibutsu" (:project app/default-db)))
    (is (= "etzhayyim-wasm-seibutsu-s3ibtsu1" (:name app/default-db)))
    (is (= "appview" (:kind app/default-db)))
    (is (= 0 (:route-count app/default-db)))
    (is (= [] (:routes app/default-db)))
    (is (= [] (:vars app/default-db)))
    (is (= true (:xrpc app/default-db)))
    (is (= "60-apps/etzhayyim-project-seibutsu/appview/etzhayyim-wasm-seibutsu-s3ibtsu1/svelte/src/routes/+page.svelte"
           (:relative-path app/default-db)))))

(deftest initialize-db-handler-test
  (testing "initialize-db-handler returns default-db regardless of prior state/event"
    (is (= app/default-db (app/initialize-db-handler {} [::app/initialize-db])))
    (is (= app/default-db (app/initialize-db-handler {:title "stale"} [::app/initialize-db])))
    (is (= app/default-db (app/initialize-db-handler nil nil)))))

(deftest dispatch-and-subscribe-roundtrip-test
  (testing "the registered ::initialize-db event and all subs wire up end-to-end"
    (rf/dispatch-sync [::app/initialize-db])
    (is (= "Seibutsu S3ibtsu1" @(rf/subscribe [::app/title])))
    (is (= "etzhayyim-project-seibutsu" @(rf/subscribe [::app/project])))
    (is (= "etzhayyim-wasm-seibutsu-s3ibtsu1" @(rf/subscribe [::app/name])))
    (is (= "appview" @(rf/subscribe [::app/kind])))
    (is (= 0 @(rf/subscribe [::app/route-count])))
    (is (= [] @(rf/subscribe [::app/routes])))
    (is (= [] @(rf/subscribe [::app/vars])))
    (is (= true @(rf/subscribe [::app/xrpc])))
    (is (= app/default-db @rf-db/app-db))))

(deftest app-view-renders-main-with-dads-container-test
  (testing "app-view is rooted at :main and wraps content in the dds-ext-container class"
    (rf/dispatch-sync [::app/initialize-db])
    (let [hiccup (app/app-view)
          container (second hiccup)]
      (is (= :main (first hiccup)))
      (is (= :div (first container)))
      (is (= "dds-ext-container" (:class (second container)))))))

(deftest facts-panel-reflects-xrpc-and-route-count-test
  (testing "facts-panel renders a dds-ext-row of 3 dds-ext-card cells; XRPC card says 'enabled' when xrpc is true"
    (rf/dispatch-sync [::app/initialize-db])
    (let [row (app/facts-panel)
          cards (vec (drop 2 row))          ; drop :div and {:class ...}
          xrpc-card (nth cards 2)]
      (is (= :div (first row)))
      (is (= "dds-ext-row" (:class (second row))))
      (is (= 3 (count cards)))
      ;; xrpc-card = [:div {:class "dds-ext-card"} [:span "XRPC"] [:strong "enabled"]]
      (is (= "enabled" (last (last xrpc-card)))))))

(deftest public-routes-panel-shows-fallback-when-empty-test
  (testing "public-routes-panel shows the no-routes fallback text when routes is empty"
    (rf/dispatch-sync [::app/initialize-db])
    (let [section (app/public-routes-panel)]
      (is (= :section (first section)))
      (is (= "dds-ext-section" (:class (second section))))
      (is (some #(and (vector? %) (= :p.muted (first %))) section)))))
